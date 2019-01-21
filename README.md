cloud-scaling
=============

Ansible role to manage applications' scaling groups on Cloud environments.

Limitations:
* AMI creation
* We have static Simple Step Scaling policy on ASG
* The alarms of scaling policies has static name, for example: `{{ asg_name }}-CPUUtilization-(High|Low)`


Requirements
------------

Ansible 2.5+

Role Variables
--------------

`cs_tags`: Global tags used on resources.
`cs_ami_create`: Enable AMI creation (not supported, only false)
`cs_lc_instance_type`: instance type to use to create Launch Configuration
`cs_lc_spot_price`: When using spot, the Spot price
`cs_lc_security_groups`: List of security groups
`cs_asg_target_group_arms`: List of target group ARNs
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

* Var file `vars/aws-asg/api.yml`

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
      cs_asg_target_group_arms:
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
            target_group_arns: "{{ cs_asg_target_group_arms }}"
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
* Make support of step scaling policies (limition of the module)
* Keep simple policies more flexible on SPEC
* Make CI tests

Feel free to contribute! =)

Author Information
------------------

DevOps Team on Linx.
