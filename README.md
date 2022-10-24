# TASK10
NW.yaml及びRDS.yaml追加

NWへ
#=================================
# ターゲットグループ作成
#=================================

  RAISETG:
    Type: "AWS::ElasticLoadBalancingV2::TargetGroup"
    Properties: 
      VpcId: !Ref RAISE
      Protocol: HTTP
      Port: 80
      HealthCheckProtocol: HTTP
      HealthCheckPath: "/"
      HealthCheckPort: "traffic-port"
      HealthyThresholdCount: 2
      UnhealthyThresholdCount: 2
      HealthCheckTimeoutSeconds: 5
      HealthCheckIntervalSeconds: 10
      Tags: 
        - Key: Name
          Value: RATG
      Matcher: 
        HttpCode: 200
      TargetGroupAttributes: 
        - Key: "deregistration_delay.timeout_seconds"
          Value: 300
        - Key: "stickiness.enabled"
          Value: false
        - Key: "stickiness.type"
          Value: lb_cookie
        - Key: "stickiness.lb_cookie.duration_seconds"
          Value: 86400
      Targets: 
        - Id: !Ref EC2
          Port: 80


# ------------------------------------------------------------#
# ALB
# ------------------------------------------------------------#      

  InternetALB: 
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties: 
      Scheme: internet-facing
      LoadBalancerAttributes: 
        - Key: deletion_protection.enabled
          Value: false
        - Key: idle_timeout.timeout_seconds
          Value: 4000
      
      SecurityGroups:
        - !Ref SECURITYGW
      Subnets: 
        - !Ref PUBLIC
        - !Ref PUBLIC2
      Tags: 
        - Key: Name
          Value: RAISEALB
  ALBListenerHTTP: 
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties: 
      Port: 80
      Protocol: HTTP
      DefaultActions: 
        - TargetGroupArn: !Ref RAISETG
          Type: forward
      LoadBalancerArn: !Ref InternetALB
