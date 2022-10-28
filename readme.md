[![N|Solid](https://github.com/satya-500/create-branch-from-tag/blob/master/iamnight.gif)](https://500apps.com)
# On-demand self-hosted AWS EC2 runner

## Example usage

```
name: do-the-job
on:
  workflow_dispatch:
jobs:
  start-aws-runner:
    name: start self-hosted ec2 runner
    runs-on: ubuntu-latest
    outputs:
      label: ${{ steps.start-ec2-runner.outputs.label }}
      ec2-instance-id: ${{ steps.start-ec2-runner.outputs.ec2-instance-id }}
    steps:
      
      - name: configure aws credentials
        uses: aws-actions/configure-aws-credentials@v1-node16
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1
      
      - name: start ec2 runner
        id: start-ec2-runner
        uses: satya-500/aws-github-runner@v1.0
        with:
          mode: start
          github-token: ${{ secrets.GH_PERSONAL_ACCESS_TOKEN }}
          ec2-image-id: ami-xxxxxxxxxxxxxxx
          ec2-instance-type: t3a.medium
          subnet-id: subnet-xxxxxxxxxxxxxxx
          security-group-id: sg-xxxxxxxxxxxxxxx
          aws-resource-tags: >
            [
              {"Key": "Name", "Value": "ec2-github-runner"}
            ]
  
  do-the-job:
    name: do the job on the runner
    needs: start-aws-runner 
    runs-on: ${{ needs.start-aws-runner.outputs.label }}
    steps:
      - name: test
        run: |
          cat /etc/os-release
  
  stop-aws-runner:
    name: stop self-hosted ec2 runner
    needs:
      - start-aws-runner
      - do-the-job
    runs-on: ubuntu-latest
    if: ${{ always() }}
    steps:
      - name: configure aws credentials
        uses: aws-actions/configure-aws-credentials@v1-node16
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1
      
      - name: stop ec2 runner
        uses: satya-500/aws-github-runner@v1.0
        with:
          mode: stop
          github-token: ${{ secrets.GH_PERSONAL_ACCESS_TOKEN }}
          label: ${{ needs.start-aws-runner.outputs.label }}
          ec2-instance-id: ${{ needs.start-aws-runner.outputs.ec2-instance-id }}
```