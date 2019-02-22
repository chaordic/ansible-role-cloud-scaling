
## Versions

* aws_application_scaling_policy.py

> https://raw.githubusercontent.com/ansible/ansible/devel/lib/ansible/modules/cloud/amazon/aws_application_scaling_policy.py

203caf2570e236e1569f7ebac738b41f51f1b02c

* ec2_scaling_policy.py

https://raw.githubusercontent.com/ansible/ansible/devel/lib/ansible/modules/cloud/amazon/ec2_scaling_policy.py

https://github.com/ansible/ansible/commit/7f98a8db1210cacebfc95474c6e5bfd8b0306e0b

7f98a8db1210cacebfc95474c6e5bfd8b0306e0b


## REf

https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html

## CMD

```bash
aws  autoscaling put-scaling-policy \
    --auto-scaling-group-name onsite-server-spotdev \
    --policy-name scale-up \
    --policy-type StepScaling \
    --adjustment-type PercentChangeInCapacity \
    --estimated-instance-warmup 90 \
    --metric-aggregation-type "Average" \
    --step-adjustments '[
                {
                    "ScalingAdjustment": 1, 
                    "MetricIntervalLowerBound": 5.0, 
                    "MetricIntervalUpperBound": 12.0
                }, 
                {
                    "ScalingAdjustment": 2, 
                    "MetricIntervalLowerBound": 12.0, 
                    "MetricIntervalUpperBound": 27.0
                }, 
                {
                    "ScalingAdjustment": 4, 
                    "MetricIntervalLowerBound": 27.0, 
                    "MetricIntervalUpperBound": 47.0
                }, 
                {
                    "ScalingAdjustment": 8, 
                    "MetricIntervalLowerBound": 47.0
                }
            ]' \
    --adjustment-type ChangeInCapacity

aws  autoscaling put-scaling-policy \
    --auto-scaling-group-name onsite-server-spotdev \
    --policy-name scale-down \
    --policy-type StepScaling \
    --metric-aggregation-type "Average" \
    --step-adjustments '[
                {
                    "ScalingAdjustment": -1, 
                    "MetricIntervalLowerBound": -10.0, 
                    "MetricIntervalUpperBound": 0.0
                }, 
                {
                    "ScalingAdjustment": -1, 
                    "MetricIntervalUpperBound": -10.0
                }
            ]' \
    --adjustment-type ChangeInCapacity



            # "Alarms": [
            #     {
            #         "AlarmName": "asg-onsite-server-spot-uppercpu", 
            #         "AlarmARN": "arn:aws:cloudwatch:us-east-1:753650202390:alarm:asg-onsite-server-spot-uppercpu"
            #     }
            # ]

```
