# Caddy A/B Deployment Strategies

## 🎯 Overview: A/B Deployment Methods

### 1. Configuration-based A/B Testing
### 2. Caddy Admin API (Dynamic Configuration)
### 3. Header-based Routing
### 4. Percentage-based Traffic Splitting
### 5. Feature Flag Integration

---

## 🔧 Method 1: Configuration-based A/B Testing

### Basic A/B Configuration
```caddyfile
{
  email admin@example.com
  order ab_testing before reverse_proxy
}

api.example.com {
  # A/B Testing based on various criteria
  
  # Version A (Stable) - 80% traffic
  @version_a {
    # Multiple conditions for A
    not header X-Beta-User "true"
    not header User-Agent "*beta*"
    not query beta=*
  }
  
  # Version B (Beta) - 20% traffic
  @version_b {
    # Beta users
    header X-Beta-User "true"
  }
  
  @version_b_rollout {
    # Gradual rollout based on IP hash
    header_regexp User-Agent ".*"
    expression {http.request.remote.host | hash | mod 100} < 20
  }
  
  handle @version_b {
    reverse_proxy app-v2:8080 {
      header_up X-Version "v2"
      header_up X-AB-Group "beta"
    }
  }
  
  handle @version_b_rollout {
    reverse_proxy app-v2:8080 {
      header_up X-Version "v2"
      header_up X-AB-Group "rollout"
    }
  }
  
  # Default to version A
  handle @version_a {
    reverse_proxy app-v1:8080 {
      header_up X-Version "v1"
      header_up X-AB-Group "stable"
    }
  }
  
  # Fallback
  reverse_proxy app-v1:8080
}
```

### NetBird A/B Example
```caddyfile
{
  email ipv9@duck.com
  order coraza_waf first
  order rate_limit before reverse_proxy
}

netbird.a.uni.net.th {
  import security_headers
  import rate_limits
  import waf_protection
  
  # A/B Testing for NetBird Management API
  @management_v2_users {
    # Beta users get v2
    header X-NetBird-Beta "true"
  }
  
  @management_v2_gradual {
    # 10% gradual rollout based on user ID hash
    header_regexp Authorization "Bearer .*"
    expression {http.request.header.authorization | hash | mod 100} < 10
  }
  
  # Version 2 (Beta Management)
  handle /api/* {
    handle @management_v2_users {
      reverse_proxy management-v2:8080 {
        header_up X-NetBird-Version "v2-beta"
      }
    }
    
    handle @management_v2_gradual {
      reverse_proxy management-v2:8080 {
        header_up X-NetBird-Version "v2-gradual"
      }
    }
    
    # Default to v1
    reverse_proxy management:8080 {
      header_up X-NetBird-Version "v1-stable"
    }
  }
  
  # Other services remain on v1
  reverse_proxy /signalexchange.SignalExchange/* h2c://signal:80
  reverse_proxy /* dashboard:80
}
```

---

## 🚀 Method 2: Caddy Admin API (Dynamic Configuration)

### Enable Admin API
```caddyfile
{
  admin 0.0.0.0:2019  # Enable admin API
  email admin@example.com
}
```

### Dynamic A/B Testing Script
```bash
#!/bin/bash
# ab-deploy.sh - Dynamic A/B deployment script

CADDY_ADMIN="http://localhost:2019"
APP_V1="app-v1:8080"
APP_V2="app-v2:8080"

# Function to get current config
get_config() {
    curl -s "$CADDY_ADMIN/config/" | jq .
}

# Function to update upstream weights
update_ab_split() {
    local v1_weight=$1
    local v2_weight=$2
    
    echo "🔄 Updating A/B split: V1=$v1_weight%, V2=$v2_weight%"
    
    # Create new configuration
    cat > /tmp/ab_config.json << EOF
{
  "apps": {
    "http": {
      "servers": {
        "srv0": {
          "listen": [":443"],
          "routes": [
            {
              "match": [{"host": ["api.example.com"]}],
              "handle": [
                {
                  "handler": "reverse_proxy",
                  "upstreams": [
                    {
                      "dial": "$APP_V1",
                      "weight": $v1_weight
                    },
                    {
                      "dial": "$APP_V2", 
                      "weight": $v2_weight
                    }
                  ],
                  "lb_policy": "weighted_round_robin"
                }
              ]
            }
          ]
        }
      }
    }
  }
}
EOF
    
    # Apply configuration
    curl -X POST "$CADDY_ADMIN/config/" \
         -H "Content-Type: application/json" \
         -d @/tmp/ab_config.json
    
    if [ $? -eq 0 ]; then
        echo "✅ A/B split updated successfully"
    else
        echo "❌ Failed to update A/B split"
        exit 1
    fi
}

# Function for gradual rollout
gradual_rollout() {
    echo "🎯 Starting gradual rollout..."
    
    # Stage 1: 90% v1, 10% v2
    update_ab_split 90 10
    echo "⏳ Stage 1: 10% traffic to v2. Monitoring for 5 minutes..."
    sleep 300
    
    # Check health metrics (implement your monitoring logic)
    if check_v2_health; then
        # Stage 2: 70% v1, 30% v2
        update_ab_split 70 30
        echo "⏳ Stage 2: 30% traffic to v2. Monitoring for 10 minutes..."
        sleep 600
        
        if check_v2_health; then
            # Stage 3: 50% v1, 50% v2
            update_ab_split 50 50
            echo "⏳ Stage 3: 50% traffic to v2. Monitoring for 15 minutes..."
            sleep 900
            
            if check_v2_health; then
                # Final: 100% v2
                update_ab_split 0 100
                echo "🎉 Rollout complete! 100% traffic to v2"
            else
                rollback_deployment
            fi
        else
            rollback_deployment
        fi
    else
        rollback_deployment
    fi
}

# Health check function
check_v2_health() {
    echo "🏥 Checking v2 health..."
    
    # Check error rate
    error_rate=$(curl -s "$CADDY_ADMIN/metrics" | grep -E "caddy_http_requests_total.*5.." | awk '{sum+=$2} END {print sum/NR}')
    
    # Check response time
    response_time=$(curl -s -w "%{time_total}" -o /dev/null http://api.example.com/health)
    
    if (( $(echo "$error_rate < 0.01" | bc -l) )) && (( $(echo "$response_time < 1.0" | bc -l) )); then
        echo "✅ v2 health check passed"
        return 0
    else
        echo "❌ v2 health check failed. Error rate: $error_rate, Response time: ${response_time}s"
        return 1
    fi
}

# Rollback function
rollback_deployment() {
    echo "🔙 Rolling back to v1..."
    update_ab_split 100 0
    echo "✅ Rollback complete"
}

# Canary deployment
canary_deploy() {
    echo "🐤 Starting canary deployment..."
    
    # 1% canary
    update_ab_split 99 1
    sleep 180  # 3 minutes
    
    if check_v2_health; then
        # 5% canary
        update_ab_split 95 5
        sleep 300  # 5 minutes
        
        if check_v2_health; then
            echo "✅ Canary successful, proceed with gradual rollout"
            gradual_rollout
        else
            rollback_deployment
        fi
    else
        rollback_deployment
    fi
}

# Blue-Green deployment
blue_green_deploy() {
    echo "🔵🟢 Blue-Green deployment..."
    
    # Warm up green (v2)
    echo "Warming up green environment..."
    curl -s http://$APP_V2/warmup > /dev/null
    
    # Health check green
    if curl -f -s http://$APP_V2/health > /dev/null; then
        echo "✅ Green environment healthy, switching traffic"
        update_ab_split 0 100
        
        # Monitor for issues
        sleep 60
        if check_v2_health; then
            echo "🎉 Blue-Green deployment successful"
        else
            echo "❌ Issues detected, rolling back"
            update_ab_split 100 0
        fi
    else
        echo "❌ Green environment unhealthy, aborting deployment"
        exit 1
    fi
}

# Main script
case "$1" in
    "gradual")
        gradual_rollout
        ;;
    "canary")
        canary_deploy
        ;;
    "blue-green")
        blue_green_deploy
        ;;
    "split")
        if [ -z "$2" ] || [ -z "$3" ]; then
            echo "Usage: $0 split <v1_percentage> <v2_percentage>"
            exit 1
        fi
        update_ab_split $2 $3
        ;;
    "rollback")
        rollback_deployment
        ;;
    "status")
        get_config | jq '.apps.http.servers.srv0.routes[0].handle[0].upstreams'
        ;;
    *)
        echo "Usage: $0 {gradual|canary|blue-green|split|rollback|status}"
        echo ""
        echo "Examples:"
        echo "  $0 gradual     # Gradual rollout"
        echo "  $0 canary      # Canary deployment"
        echo "  $0 blue-green  # Blue-green deployment"
        echo "  $0 split 80 20 # 80% v1, 20% v2"
        echo "  $0 rollback    # Rollback to v1"
        echo "  $0 status      # Show current split"
        exit 1
        ;;
esac
```

---

## 📊 Method 3: Advanced A/B with Feature Flags

### Feature Flag Integration
```caddyfile
{
  email admin@example.com
  order feature_flags before reverse_proxy
}

api.example.com {
  # Feature flag based routing
  @feature_enabled {
    # Check feature flag service
    http {
      url "http://feature-flags:8080/check"
      method POST
      body '{"user_id": "{http.request.header.x-user-id}", "feature": "new_api_v2"}'
      headers {
        Content-Type "application/json"
      }
    }
    status 200
  }
  
  handle @feature_enabled {
    reverse_proxy app-v2:8080 {
      header_up X-Feature-Flag "enabled"
    }
  }
  
  # Default to v1
  reverse_proxy app-v1:8080 {
    header_up X-Feature-Flag "disabled"
  }
}
```

### LaunchDarkly Integration Example
```caddyfile
api.example.com {
  @launchdarkly_enabled {
    # Call LaunchDarkly API
    expression `{
      http_request("GET", "https://sdk.launchdarkly.com/sdk/evalx/user/" + 
                   {http.request.header.x-user-id} + "/features/api-v2", 
                   {"Authorization": "Bearer " + {env.LAUNCHDARKLY_TOKEN}}).status == 200
    }`
  }
  
  handle @launchdarkly_enabled {
    reverse_proxy app-v2:8080
  }
  
  reverse_proxy app-v1:8080
}
```

---

## 🎛️ Method 4: Percentage-based Traffic Splitting

### Hash-based Splitting
```caddyfile
api.example.com {
  # Consistent hash-based splitting
  @version_a {
    expression {http.request.remote.host | hash | mod 100} < 80
  }
  
  @version_b {
    expression {http.request.remote.host | hash | mod 100} >= 80
  }
  
  handle @version_b {
    reverse_proxy app-v2:8080 {
      header_up X-AB-Version "v2"
    }
  }
  
  handle @version_a {
    reverse_proxy app-v1:8080 {
      header_up X-AB-Version "v1"
    }
  }
}
```

### User ID-based Splitting
```caddyfile
api.example.com {
  # Split based on user ID hash
  @user_group_a {
    header_regexp Authorization "Bearer (.*)"
    expression {re.authorization.1 | hash | mod 100} < 70
  }
  
  @user_group_b {
    header_regexp Authorization "Bearer (.*)"
    expression {re.authorization.1 | hash | mod 100} >= 70
  }
  
  handle @user_group_b {
    reverse_proxy app-v2:8080
  }
  
  handle @user_group_a {
    reverse_proxy app-v1:8080
  }
}
```

---

## 🚀 Method 5: NetBird-specific A/B Deployment

### Complete NetBird A/B Setup
```yaml
# docker-compose-ab.yml
version: "3.8"

services:
  caddy:
    image: itarun/caddy-netbird:latest
    restart: unless-stopped
    networks: [netbird]
    ports:
      - '443:443'
      - '443:443/udp'
      - '80:80'
      - '2019:2019'  # Admin API
    volumes:
      - netbird_caddy_data:/data
      - netbird_caddy_config:/config
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./ab-scripts:/scripts
    environment:
      - CADDY_ADMIN_LISTEN=0.0.0.0:2019

  # Management V1 (Stable)
  management-v1:
    image: netbirdio/management:v0.24.0
    restart: unless-stopped
    networks: [netbird]
    volumes:
      - netbird-mgmt-v1:/var/lib/netbird
      - ./management.json:/etc/netbird/management.json
    command: ["--port", "80", "--log-level", "info"]

  # Management V2 (Beta)
  management-v2:
    image: netbirdio/management:v0.25.0-beta
    restart: unless-stopped
    networks: [netbird]
    volumes:
      - netbird-mgmt-v2:/var/lib/netbird
      - ./management-v2.json:/etc/netbird/management.json
    command: ["--port", "80", "--log-level", "info"]

  # A/B Testing Controller
  ab-controller:
    build: ./ab-controller
    networks: [netbird]
    volumes:
      - ./ab-scripts:/scripts
    environment:
      - CADDY_ADMIN_URL=http://caddy:2019
      - PROMETHEUS_URL=http://prometheus:9090
    depends_on:
      - caddy
      - prometheus

volumes:
  netbird-mgmt-v1:
  netbird-mgmt-v2:
  netbird_caddy_data:
  netbird_caddy_config:

networks:
  netbird:
```

### NetBird A/B Caddyfile
```caddyfile
{
  admin 0.0.0.0:2019
  email ipv9@duck.com
  order coraza_waf first
  order rate_limit before reverse_proxy
}

netbird.a.uni.net.th {
  import security_headers
  import rate_limits
  import waf_protection
  
  # Health check
  respond /health "OK" 200
  
  # A/B Testing for Management API
  handle /api/* {
    # Beta users (opt-in)
    @beta_users {
      header X-NetBird-Beta "true"
    }
    
    # Gradual rollout (based on user hash)
    @gradual_rollout {
      header_regexp Authorization "Bearer (.*)"
      expression {re.authorization.1 | hash | mod 100} < {env.ROLLOUT_PERCENTAGE:0}
    }
    
    # Employee testing
    @employee_testing {
      remote_ip 192.168.1.0/24  # Internal network
      header X-NetBird-Employee "true"
    }
    
    handle @beta_users {
      reverse_proxy management-v2:80 {
        header_up X-NetBird-Version "v2-beta"
        header_up X-AB-Group "beta"
      }
    }
    
    handle @employee_testing {
      reverse_proxy management-v2:80 {
        header_up X-NetBird-Version "v2-employee"
        header_up X-AB-Group "employee"
      }
    }
    
    handle @gradual_rollout {
      reverse_proxy management-v2:80 {
        header_up X-NetBird-Version "v2-rollout"
        header_up X-AB-Group "rollout"
      }
    }
    
    # Default to stable v1
    reverse_proxy management-v1:80 {
      header_up X-NetBird-Version "v1-stable"
      header_up X-AB-Group "stable"
    }
  }
  
  # Other services (no A/B testing)
  reverse_proxy /signalexchange.SignalExchange/* h2c://signal:80
  reverse_proxy /* dashboard:80
}
```

### A/B Testing Controller
```python
# ab-controller/app.py
import os
import time
import requests
import logging
from datetime import datetime, timedelta

class NetBirdABController:
    def __init__(self):
        self.caddy_admin = os.getenv('CADDY_ADMIN_URL', 'http://caddy:2019')
        self.prometheus_url = os.getenv('PROMETHEUS_URL', 'http://prometheus:9090')
        self.rollout_percentage = 0
        
    def update_rollout_percentage(self, percentage):
        """Update rollout percentage via environment variable"""
        config = {
            "apps": {
                "http": {
                    "servers": {
                        "srv0": {
                            "routes": [
                                {
                                    "match": [{"host": ["netbird.a.uni.net.th"]}],
                                    "handle": [{
                                        "handler": "vars",
                                        "ROLLOUT_PERCENTAGE": str(percentage)
                                    }]
                                }
                            ]
                        }
                    }
                }
            }
        }
        
        response = requests.post(f"{self.caddy_admin}/config/", json=config)
        if response.status_code == 200:
            self.rollout_percentage = percentage
            logging.info(f"Rollout percentage updated to {percentage}%")
            return True
        else:
            logging.error(f"Failed to update rollout: {response.text}")
            return False
    
    def get_metrics(self):
        """Get metrics from Prometheus"""
        queries = {
            'error_rate_v1': 'rate(caddy_http_requests_total{ab_group="stable",status=~"5.."}[5m])',
            'error_rate_v2': 'rate(caddy_http_requests_total{ab_group="rollout",status=~"5.."}[5m])',
            'response_time_v1': 'histogram_quantile(0.95, caddy_http_request_duration_seconds{ab_group="stable"})',
            'response_time_v2': 'histogram_quantile(0.95, caddy_http_request_duration_seconds{ab_group="rollout"})'
        }
        
        metrics = {}
        for name, query in queries.items():
            response = requests.get(f"{self.prometheus_url}/api/v1/query", 
                                  params={'query': query})
            if response.status_code == 200:
                data = response.json()
                if data['data']['result']:
                    metrics[name] = float(data['data']['result'][0]['value'][1])
                else:
                    metrics[name] = 0
            else:
                metrics[name] = 0
        
        return metrics
    
    def is_v2_healthy(self):
        """Check if v2 is performing well"""
        metrics = self.get_metrics()
        
        # Health criteria
        error_rate_threshold = 0.01  # 1%
        response_time_threshold = 1.0  # 1 second
        
        v2_error_rate = metrics.get('error_rate_v2', 0)
        v2_response_time = metrics.get('response_time_v2', 0)
        
        if v2_error_rate <= error_rate_threshold and v2_response_time <= response_time_threshold:
            logging.info(f"V2 healthy - Error rate: {v2_error_rate:.3f}, Response time: {v2_response_time:.3f}s")
            return True
        else:
            logging.warning(f"V2 unhealthy - Error rate: {v2_error_rate:.3f}, Response time: {v2_response_time:.3f}s")
            return False
    
    def gradual_rollout(self):
        """Perform gradual rollout"""
        stages = [5, 10, 25, 50, 75, 100]
        
        for stage in stages:
            logging.info(f"Starting rollout stage: {stage}%")
            
            if not self.update_rollout_percentage(stage):
                logging.error("Failed to update rollout percentage")
                self.rollback()
                return False
            
            # Monitor for 10 minutes
            monitoring_time = 600  # 10 minutes
            check_interval = 30   # 30 seconds
            
            for i in range(0, monitoring_time, check_interval):
                time.sleep(check_interval)
                
                if not self.is_v2_healthy():
                    logging.error(f"V2 unhealthy during stage {stage}%, rolling back")
                    self.rollback()
                    return False
                
                logging.info(f"Stage {stage}% - Health check passed ({i+check_interval}/{monitoring_time}s)")
            
            logging.info(f"Stage {stage}% completed successfully")
        
        logging.info("🎉 Gradual rollout completed successfully!")
        return True
    
    def rollback(self):
        """Rollback to 0% v2 traffic"""
        logging.info("🔙 Rolling back to v1...")
        self.update_rollout_percentage(0)
    
    def canary_deploy(self):
        """Canary deployment (1% traffic)"""
        logging.info("🐤 Starting canary deployment...")
        
        if not self.update_rollout_percentage(1):
            return False
        
        # Monitor canary for 30 minutes
        time.sleep(1800)
        
        if self.is_v2_healthy():
            logging.info("✅ Canary successful, proceeding with gradual rollout")
            return self.gradual_rollout()
        else:
            logging.error("❌ Canary failed, rolling back")
            self.rollback()
            return False

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    controller = NetBirdABController()
    
    # Start canary deployment
    controller.canary_deploy()
```

---

## 📊 Monitoring และ Alerting

### Prometheus Metrics
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'caddy-ab'
    static_configs:
      - targets: ['caddy:2019']
    metrics_path: /metrics

rule_files:
  - "ab_alerts.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

### Alert Rules
```yaml
# ab_alerts.yml
groups:
  - name: ab_testing
    rules:
      - alert: HighErrorRateV2
        expr: rate(caddy_http_requests_total{ab_group="rollout",status=~"5.."}[5m]) > 0.01
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High error rate in A/B test version 2"
          description: "Version 2 error rate is {{ $value }} which is above threshold"

      - alert: SlowResponseTimeV2
        expr: histogram_quantile(0.95, caddy_http_request_duration_seconds{ab_group="rollout"}) > 1.0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Slow response time in A/B test version 2"
```

## 🚀 Deployment Commands

### การใช้งาน A/B Controller
```bash
# Start A/B deployment
docker-compose -f docker-compose-ab.yml up -d

# Monitor rollout
docker-compose logs -f ab-controller

# Manual rollout control
docker-compose exec ab-controller python -c "
from app import NetBirdABController
controller = NetBirdABController()
controller.gradual_rollout()
"

# Emergency rollback
docker-compose exec ab-controller python -c "
from app import NetBirdABController
controller = NetBirdABController()
controller.rollback()
"

# Check current status
curl -s http://localhost:2019/config/ | jq '.apps.http.servers.srv0.routes[0].handle[0].vars'
```

นี่คือวิธีการทำ A/B deployment ด้วย Caddy ที่ครอบคลุมและยืดหยุ่น เหมาะสำหรับทุกระดับตั้งแต่ simple configuration จนถึง automated deployment pipeline 