# SMTP relay service

This helm-chart installs [MailDev](https://github.com/maildev/maildev).

Its mainl usage is to provide you with a SMTP relay service inside Kubernetes,
so other apps can rely on it to send mails externally.

Inside the Namespace where it is deployed, an SMTP service is available: `maildev-smtp:1025`.

MailDev also provides a Web interface, it can be disabled/enabled at discretion.

Also note that mails do not persist after reboot. Everytime MailDev starts, it starts from scratch,
even if the `/tmp/maildev` folder, where MailDev stores mails, is persisted.

## Sources code

MailDev source code can be found here: https://github.com/maildev/maildev.

## Known issue with Env Vars

GitHub issue: https://github.com/maildev/maildev/issues/315

So, not every option can be configured using env vars, so, a few options are hardcoded in the deployment:
`["--verbose", "--outgoing-secure", "--auto-relay"]`

## Configuration

Table with the most relevant parameters for MailDev.
Not listing here the more general paramaters such as tolerations, nodeSelectors, etc.

| Parameter                     | Description                                                                                       | Default                                     |
|------------------------------:|:--------------------------------------------------------------------------------------------------|:--------------------------------------------|
| **outgoingRelay.host**       | SMTP Relay host, `MAILDEV_OUTGOING_HOST`.                                                         | ``                                          |
| **outgoingRelay.port**       | SMTP Relay port, `MAILDEV_OUTGOING_PORT`.                                                         | ``                                          |
| **outgoingRelay.user**       | SMTP Relay user, `MAILDEV_OUTGOING_USER`.                                                         | ``                                          |
| **outgoingRelay.pass**       | SMTP Relay password, `MAILDEV_OUTGOING_PASS`.                                                     | ``                                          |
| **outgoingRelay.secure**     | Use SMTP SSL for outgoing emails, `MAILDEV_OUTGOING_SECURE`.                                      | `true`. Hardcoded in the deployment due to a bug. |
| **ports.smtp**                | Port where the SMTP service is listening. (Irrelevant for OCP/K8S), `MAILDEV_SMTP_PORT`.          | `1025`                                      |
| **ports.web**                 | Port where the Web interface service is listening. (Irrelevant for OCP/K8S), `MAILDEV_WEB_PORT`.  | `1080`                                      |
| **web.disable**               | Disable Web interface. `MAILDEV_DISABLE_WEB`.                                                     | `false`                                     |
| **web.user**                  | Web interface user, `MAILDEV_WEB_USER`.                                                           | `admin`                                     |
| **web.pass**                  | Web interface password, `MAILDEV_WEB_PASS`.                                                       | ``                                          |
| **https.enabled**             | Switch from http to https protocol, `MAILDEV_HTTPS`.                                              | `false`                                     |
| **https.key**                 | The file path to the ssl private key, `MAILDEV_HTTPS_KEY`.                                        |                                             |
| **https.cert**                | The file path to the ssl cert file, `MAILDEV_HTTPS_CERT`.                                         |                                             |
| **incoming.user**             | SMTP user for incoming emails, `MAILDEV_INCOMING_USER`.                                           |                                             |
| **incoming.pass**             | SMTP password for incoming emails, `MAILDEV_INCOMING_PASS`.                                       |                                             |

## Test it

Open a shell into the pod.
```bash
kubectl exec --stdin --tty $(kubectl get pod -l "app.kubernetes.io/instance=maildev" -o name) -- sh
```

Create dummy mail.txt file.
```bash
cat <<EOF >> mail.txt
From: Test Maildev <test@maildev.com>
To: Nikola Tesla Tudela <niko@tesla.com>
Subject: Test mail from maildev
Date: Fri, 17 Nov 2020 11:26:16

Dear Joe,
Welcome to this example email. What a lovely day.
Cheers!!
EOF
```

Send the mail with curl:
```bash
apk add --no-cache curl && curl smtp://maildev-smtp:1025 --mail-from test@maildev.com --mail-rcpt niko@tesla.com --upload-file ./mail.txt
```

Check the logs and see if the mail has been delivered.
```bash
kubectl logs $(kubectl get pod -l "app.kubernetes.io/instance=maildev" -o name)
```
Output should be sth. like:
```bash
Temporary directory created at /tmp/maildev
Temporary directory created at /tmp/maildev/1
MailDev outgoing SMTP Server smtp.gmail.com:465 (user:test@example.com, pass:******, secure:yes)
Auto-Relay mode on
MailDev webapp running at http://0.0.0.0:1080
MailDev SMTP Server running at 0.0.0.0:1025
Saving email: Test mail from maildev, id: 3ZhYnk5q
MailDev outgoing SMTP Server smtp.gmail.com:465 (user:test@example.com, pass:******, secure:yes)
Mail Delivered:  Test mail from maildev
```

Alternatively forward your local port to the pod.
```bash
kubectl port-forward $(kubectl get pod -l "app.kubernetes.io/instance=maildev" -o name) 1080
```

Check your mail inbox ;)
