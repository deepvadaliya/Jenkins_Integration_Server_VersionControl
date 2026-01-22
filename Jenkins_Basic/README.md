# Jenkins without Elastic IP Allocation

Jenkins slows after EC2 restart without Elastic IP because the public IP changes, but Jenkins config retains the old IP, causing redirects, polling loops, and UI delays. Fix by updating the config XML or UI, and prevent via Elastic IP.

Immediate Fix: Update Config
SSH into the instance (ec2-user@new-public-ip).

Edit the Jenkins URL file:

sudo -i
cd /var/lib/jenkins
vi jenkins.model.JenkinsLocationConfiguration.xml
