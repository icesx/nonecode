gitlab
==============
##hosts and hostname
you must set your local's hostname is xx.net and add the correct ip with xx.net to hosts

##错误处理

1. had an error: Acme::Client::Error::Timeout: Acme::Client::Error::Timeout

域名xx.net无法访问到，建议去掉ssl

##install
1. Install and configure the necessary dependencies
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates
Next, install Postfix to send notification emails. If you want to use another solution to send emails please skip this step and configure an external SMTP server after GitLab has been installed.

sudo apt-get install -y postfix
During Postfix installation a configuration screen may appear. Select 'Internet Site' and press enter. Use your server's external DNS for 'mail name' and press enter. If additional screens appear, continue to press enter to accept the defaults.

2. Add the GitLab package repository and install the package
Add the GitLab package repository.

curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
Next, install the GitLab package. Change https://gitlab.example.com to the URL at which you want to access your GitLab instance. Installation will automatically configure and start GitLab at that URL.

For https:// URLs GitLab will automatically request a certificate with Let's Encrypt, which requires inbound HTTP access and a valid hostname. You can also use your own certificate or just use http://.

sudo EXTERNAL_URL="https://gitlab.example.com" apt-get install gitlab-ee
3. Browse to the hostname and login
On your first visit, you'll be redirected to a password reset screen. Provide the password for the initial administrator account and you will be redirected back to the login screen. Use the default account's username root to login.

See our documentation for detailed instructions on installing and configuration.

4. Set up your communication preferences
Visit our email subscription preference center to let us know when to communicate with you. We have an explicit email opt-in policy so you have complete control over what and how often we send you emails.

Twice a month, we send out the GitLab news you need to know, including new features, integrations, docs, and behind the scenes stories from our dev teams. For critical security updates related to bugs and system performance, sign up for our dedicated security newsletter.

IMPORTANT NOTE: If you do not opt-in to the security newsletter, you will not receive security alerts.