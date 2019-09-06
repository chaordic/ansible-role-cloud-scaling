cloud-scaling
=============

Ansible role to manage applications' scaling groups on Cloud environments.

Limitations:
* AMI creation
* We have static Simple Step Scaling policy on ASG
* The alarms of scaling policies has static name, for example: `{{ asg_name }}-CPUUtilization-(High|Low)`


Requirements
------------

- Ansible 2.5+
- awscli 1.16+

Role Variables
--------------

`cs_tags`: Global tags used on resources.

`cs_ami_create`: Enable AMI creation (not supported, only false)

`cs_lc_instance_type`: instance type to use to create Launch Configuration

`cs_lc_spot_price`: When using spot, the Spot price

`cs_lc_security_groups`: List of security groups

`cs_asg_target_group_arns`: List of target group ARNs

`cs_asg_availability_zones`: List of availability zones to be used on ASG

`cs_asg_vpc_zone_identifier`: List of subnets to be used on ASG

`cs_asg_termination_policies`: List of termination policies used on ASG

`cs_asg_notification_topic`: ARN of topic to publish alerts

`cs_asg_notification_types`: List of alerts to be triggered

`cs_scaling_groups`: Scaling policies (ASGs) to be created

Dependencies
------------

* AWS account
* AWS env vars exported
* AWS programatic permissions

Example Playbook
----------------

* Var file `vars/aws-asg/api.yml` with **simple scaling policy**

      ---# vars/aws-asg/api.yml
      ### Global
      cs_tags:
        team: myTeam
        env: prod
        role: api

      ### Global CS AMI
      cs_ami_create: false

      ### Global CS LC
      cs_lc_instance_type: c5.4xlarge
      cs_lc_spot_price: 0.68
      cs_lc_security_groups:
        - apps
        - api

      ### Global CS ASG
      cs_asg_target_group_arns:
        - arn:aws:elasticloadbalancing:us-east-1:000000000000:targetgroup/tg-api/abcdef0123456789

      cs_asg_availability_zones:
        - us-east-1d

      cs_asg_vpc_zone_identifier:
        - subnet-abcdef0123456789

      cs_asg_termination_policies:
        - OldestLaunchConfiguration
        - ClosestToNextInstanceHour
        - OldestInstance
        - Default

      cs_asg_notification_topic: arn:aws:sns:us-east-1:000000000000:MyTeam
      cs_asg_notification_types:
        - autoscaling:EC2_INSTANCE_LAUNCH_ERROR
        - autoscaling:EC2_INSTANCE_TERMINATE_ERROR

      ### Scaling Groups
      cs_scaling_groups:
        - asg_name: api-devspot
          lc_name: "api-201901211852-devspot-c5.4xlarge"
          lc_spec:
            instance_type: "{{ cs_lc_instance_type }}"
            security_groups: "{{ cs_lc_security_groups }}"
            instance_monitoring: yes
            instance_profile_name: instance-role-api
            ebs_optimized: yes
            image_id: ami-abcdef0123456789
            spot_price: "{{ cs_lc_spot_price }}"
          asg_spec:
            min_size: 0
            max_size: 5
            desired_capacity: 0
            health_check_type: "ELB"
            target_group_arns: "{{ cs_asg_target_group_arns }}"
            availability_zones: "{{ cs_asg_availability_zones }}"
            vpc_zone_identifier: "{{ cs_asg_vpc_zone_identifier }}"
            termination_policies: "{{ cs_asg_termination_policies }}"
            notification_topic: "{{ cs_asg_notification_topic }}"
            notification_types: "{{ cs_asg_notification_types }}"
            tags:
              - Name: api-devspot
                propagate_at_launch: yes
              - team: "{{ cs_tags.team }}"
                propagate_at_launch: yes
              - env: "dev"
                propagate_at_launch: yes
          metrics_spec:
            cpu_high_threshold: 45.0
            cpu_low_threshold: 20.0
          scaling_policies_type: simple   # only one supported
          scaling_policies:
            - name: "scale-up"
              adjustment_type: "ChangeInCapacity"
              scaling_adjustment: +1
              min_adjustment_step: 1
              cooldown: 60
            - name: "scale-down"
              adjustment_type: "ChangeInCapacity"
              scaling_adjustment: -1
              min_adjustment_step: 1
              cooldown: 300

* Var file `vars/aws-asg/api.yml` with **step scaling policy**

      ---# vars/aws-asg/api.yml
      ### Global
      cs_tags:
        team: myTeam
        env: prod
        role: api

      ### Global CS AMI
      cs_ami_create: false

      ### Global CS LC
      cs_lc_instance_type: c5.4xlarge
      cs_lc_spot_price: 0.68
      cs_lc_security_groups:
        - apps
        - api

      ### Global CS ASG
      cs_asg_target_group_arns:
        - arn:aws:elasticloadbalancing:us-east-1:000000000000:targetgroup/tg-api/abcdef0123456789

      cs_asg_availability_zones:
        - us-east-1d

      cs_asg_vpc_zone_identifier:
        - subnet-abcdef0123456789

      cs_asg_termination_policies:
        - OldestLaunchConfiguration
        - ClosestToNextInstanceHour
        - OldestInstance
        - Default

      cs_asg_notification_topic: arn:aws:sns:us-east-1:000000000000:MyTeam
      cs_asg_notification_types:
        - autoscaling:EC2_INSTANCE_LAUNCH_ERROR
        - autoscaling:EC2_INSTANCE_TERMINATE_ERROR

      ### Scaling Groups
      cs_scaling_groups:
        - asg_name: api-devspot
          lc_name: "api-201901211852-devspot-c5.4xlarge"
          lc_spec:
            instance_type: "{{ cs_lc_instance_type }}"
            security_groups: "{{ cs_lc_security_groups }}"
            instance_monitoring: yes
            instance_profile_name: instance-role-api
            ebs_optimized: yes
            image_id: ami-abcdef0123456789
            spot_price: "{{ cs_lc_spot_price }}"
            vpc_id: "{{ cs_lc_vpc_id }}"

          asg_spec:
            min_size: 0
            max_size: 5
            desired_capacity: 0
            health_check_type: "ELB"
            target_group_arns: "{{ cs_asg_target_group_arns }}"
            availability_zones: "{{ cs_asg_availability_zones }}"
            vpc_zone_identifier: "{{ cs_asg_vpc_zone_identifier }}"
            termination_policies: "{{ cs_asg_termination_policies }}"
            notification_topic: "{{ cs_asg_notification_topic }}"
            notification_types: "{{ cs_asg_notification_types }}"
            tags:
              - Name: api-devspot
                propagate_at_launch: yes
              - team: "{{ cs_tags.team }}"
                propagate_at_launch: yes
              - env: "dev"
                propagate_at_launch: yes
          scaling_policies_type: step
          scaling_policies_spec:
            - name: scale-up
              policy_type: StepScaling
              adjustment_type: PercentChangeInCapacity
              metric_aggregation_type: Average
              estimated_warmup: 90
              alarm_spec:
                metric: "CPUUtilization"
                name: CPUUtilization-High
                statistic: "Average"
                comparison: ">="
                threshold: 50
                minutes: 3
              step_adjustments:
                  - "ScalingAdjustment": 1
                    "MetricIntervalLowerBound": 0
                    "MetricIntervalUpperBound": 10
                  - "ScalingAdjustment": 2
                    "MetricIntervalLowerBound": 10
                    "MetricIntervalUpperBound": 20
                  - "ScalingAdjustment": 4
                    "MetricIntervalLowerBound": 20
                    "MetricIntervalUpperBound": 35
                  - "ScalingAdjustment": 8
                    "MetricIntervalLowerBound": 35
            - name: scale-down
              state: present
              policy_type: StepScaling
              adjustment_type: ChangeInCapacity
              metric_aggregation_type: Average
              estimated_warmup: 120
              alarm_spec:
                metric: "CPUUtilization"
                name: CPUUtilization-Low
                statistic: "Average"
                comparison: "<="
                threshold: 30
                minutes: 10
              step_adjustments:
                  - "ScalingAdjustment": -1
                    "MetricIntervalLowerBound": -10
                    "MetricIntervalUpperBound": 0
                  - "ScalingAdjustment": -2
                    "MetricIntervalUpperBound": -10


* Var file `vars/aws-asg/api.yml` with **mixed instances policy**

      ---# vars/aws-asg/api.yml
      ### Global
      cs_tags:
        team: myTeam
        env: prod
        role: api

      ### Global CS AMI
      cs_ami_create: false

      # Global CS LT
      cs_lt_security_groups:
      - sg-0f485a3637b7681f5
      - sg-1z521x3670b3301f2

      ### Global CS ASG
      cs_asg_target_group_arns:
        - arn:aws:elasticloadbalancing:us-east-1:000000000000:targetgroup/tg-api/abcdef0123456789

      cs_asg_instance_types:
        - c5.large
        - c5d.large
        - m5.large
        - m5d.large

      cs_asg_availability_zones:
        - us-east-1d

      cs_asg_vpc_zone_identifier:
        - subnet-abcdef0123456789

      cs_asg_termination_policies:
        - OldestLaunchTemplate
        - ClosestToNextInstanceHour
        - OldestInstance
        - AllocationStrategy
        - Default

      cs_asg_notification_topic: arn:aws:sns:us-east-1:000000000000:MyTeam
      cs_asg_notification_types:
        - autoscaling:EC2_INSTANCE_LAUNCH_ERROR
        - autoscaling:EC2_INSTANCE_TERMINATE_ERROR

      ### Scaling Groups
      cs_scaling_groups:
        - asg_name: api-devfleet
          lt_name: "api-fleet-201901211852"
          lt_spec:
            security_groups: "{{ cs_lt_security_groups }}"
            instance_monitoring: true
            instance_profile_name: instance-role-api
            ebs_optimized: yes
            image_id: "{{ latest_ami  }}"

          asg_mixed_instances_policy:
            on_demand_base_capacity: 1
            on_demand_percentage_above_base_capacity: 1
            spot_allocation_strategy: capacity-optimized
            instance_types: "{{ cs_asg_instance_types }}"

          asg_spec:
            min_size: 0
            max_size: 5
            desired_capacity: 0
            health_check_type: "ELB"
            target_group_arns: "{{ cs_asg_target_group_arns }}"
            availability_zones: "{{ cs_asg_availability_zones }}"
            vpc_zone_identifier: "{{ cs_asg_vpc_zone_identifier }}"
            termination_policies: "{{ cs_asg_termination_policies }}"
            notification_topic: "{{ cs_asg_notification_topic }}"
            notification_types: "{{ cs_asg_notification_types }}"
            tags:
              - Name: api-devspot
                propagate_at_launch: yes
              - team: "{{ cs_tags.team }}"
                propagate_at_launch: yes
              - env: "dev"
                propagate_at_launch: yes

* Playbook `aws-asg-api.yml`

      ---# aws-asg-api.yml
      - name: Cloud ASG - API
        hosts: 127.0.0.1
        connection: local
        gather_facts: true
        vars_files:
          - vars/aws-asg/api.yml

      roles:
         - role: cloud-scaling.chaordic

* Running the playbook

      ansible-playbook aws-asg-api.yml -D -vvv


License
-------

GPLv3

Contributing
------------

We have some TODOs, like:

* Make role idempotent on modules `ec2_asg` and `ec2_metric_alarm`
* Make support of step scaling policies (limitation of the module)
* Keep simple policies more flexible on SPEC
* Make CI tests

Feel free to contribute! =)

Author Information
------------------

DevOps Team on Linx.
