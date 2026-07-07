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

## Configuration

Table with the most relevant parameters for MailDev.
Not listing here the more general paramaters such as tolerations, nodeSelectors, etc.

| Parameter                            | Description                                                                                       | Default                                     |
|-------------------------------------:|:--------------------------------------------------------------------------------------------------|:--------------------------------------------|
| **image.tag**                        | maildev image tag; defaults to the chart appVersion. Set to a 3.x tag to opt in (see below).     | `""` (appVersion)                           |
| **probePath**                        | HTTP path for the liveness/readiness probes. Use `/` for maildev 3.x (which removed `/healthz`).  | `/healthz`                                  |
| **verbose**                          | Enable verbose logging, `--verbose`.                                                             | `false`                                     |
| **extraArgs**                        | Extra CLI args appended after the chart-generated flags, for options not modelled below.         | `[]`                                        |
| **extraEnv**                         | Extra container env vars (list of `name`/`value` or `valueFrom` entries), for options not modelled below. | `[]`                              |
| **outgoingRelay.host**               | SMTP Relay host, `MAILDEV_OUTGOING_HOST`. Only set when defined.                                  | ``                                          |
| **outgoingRelay.port**               | SMTP Relay port, `MAILDEV_OUTGOING_PORT`. Only set when defined.                                  | ``                                          |
| **outgoingRelay.user**               | SMTP Relay user, `MAILDEV_OUTGOING_USER`. Only set when defined.                                  | ``                                          |
| **outgoingRelay.pass**               | SMTP Relay password, `MAILDEV_OUTGOING_PASS`. Only set when defined.                              | ``                                          |
| **outgoingRelay.secure**             | Use SMTP SSL for outgoing emails, `--outgoing-secure`.                                            | `false`                                     |
| **outgoingRelay.autoRelay.enabled**  | Relay all incoming emails to the outgoing relay, `--auto-relay`.                                  | `false`                                     |
| **outgoingRelay.autoRelay.receiver** | Relay to a single receiver instead of the original recipient, `MAILDEV_AUTO_RELAY`.               | ``                                          |
| **outgoingRelay.autoRelay.rules**    | Auto relay rules rendered to `auto-relay-rules.json`, `MAILDEV_AUTO_RELAY_RULES`.                 | `[{allow: '*'}]`                            |
| **ports.smtp**                       | Port where the SMTP service is listening. (Irrelevant for OCP/K8S), `MAILDEV_SMTP_PORT`.          | `1025`                                      |
| **ports.web**                        | Port where the Web interface service is listening. (Irrelevant for OCP/K8S), `MAILDEV_WEB_PORT`.  | `1080`                                      |
| **web.disable**                      | Disable Web interface, `--disable-web`.                                                           | `false`                                     |
| **web.user**                         | Web interface user, `MAILDEV_WEB_USER`. Only set when defined.                                    | ``                                          |
| **web.pass**                         | Web interface password, `MAILDEV_WEB_PASS`. Only set when defined.                               | ``                                          |
| **https.enabled**             | Switch from http to https protocol, `MAILDEV_HTTPS`.                                              | `false`                                     |
| **https.key**                 | The file path to the ssl private key, `MAILDEV_HTTPS_KEY`.                                        |                                             |
| **https.cert**                | The file path to the ssl cert file, `MAILDEV_HTTPS_CERT`.                                         |                                             |
| **incoming.user**             | SMTP user for incoming emails, `MAILDEV_INCOMING_USER`.                                           |                                             |
| **incoming.pass**             | SMTP password for incoming emails, `MAILDEV_INCOMING_PASS`.                                       |                                             |

## Trying maildev 3.x

The chart defaults to the latest stable maildev (2.x). maildev 3.x is a
rewrite and currently only ships a release candidate, so it is not the
default. It removed the `/healthz` endpoint the probes use, so opting in
requires pointing the probes at `/`:

```yaml
image:
  tag: "3.0.0-rc.1"
probePath: "/"
```

The chart's flags and env vars are otherwise compatible with 3.x.

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
