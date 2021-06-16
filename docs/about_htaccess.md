# About .htaccess

k8sでデプロイしたappcheサーバー内の.htaccessについて
httpsリダイレクトさせる場合の例
参考、引用：https://spltech.co.uk/how-to-do-a-redirect-from-http-to-https-with-gke-ingress/
```
<IfModule mod_rewrite.c>
 RewriteEngine On
 RewriteCond %{HTTP_USER_AGENT}  !^GoogleHC\/(.*)$
 RewriteCond %{HTTP_USER_AGENT}  !^kube-probe\/(.*)$
 RewriteCond %{HTTP:X-Forwarded-Proto} !https
 RewriteCond %{HTTP_HOST} !localhost
 RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
 </IfModule>
```

```
 RewriteCond %{HTTP_USER_AGENT}  !^GoogleHC\/(.*)$
 RewriteCond %{HTTP_USER_AGENT}  !^kube-probe\/(.*)
```
KubernetesとGKELBの定期的なヘルスチェックが301リダイレクトされると誤作動を起こす。
引用元では、probeの変更を試みたが、できなかったとのこと。

```
 RewriteCond %{HTTP:X-Forwarded-Proto} !https
```
`%{HTTP:X-Forwarded-Proto}` ではなく、 `%{HTTP}` にするとリダイレクトループが起きる。
原因は、ロードバランサーなどのリバースプロキシのssl.conf内で、全てのポート443がポート80にプロキシされるため、永遠にhttps = onになることがため。
X-Forwarded-Protを見ることで、リクエストがhttps or httpを判別できる。AWS推奨の書き方(https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/configuring-https-httpredirect.html)。
参考：
https://ikuty.com/2016/07/02/aws-htaccess-without-redirectloop/
https://stackoverflow.com/questions/20363051/aws-elasticbeanstalk-single-instance-force-ssl-redirect-loop

わかっていないこと：
```
 RewriteCond %{HTTP:X-Forwarded-Proto} !https
```
はGoogleでも推奨の書き方？
AWSやGCP特有の現象？
App Engineとかでも起きる？（ssl.conf内で443がポート80にプロキシする記述を見た気がする）