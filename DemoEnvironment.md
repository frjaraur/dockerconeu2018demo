
## Demo Environment for DockerCon EU 2018
~~~
hopla-ucp-manager
34.251.95.192 hopla-ucp-manager
~~~

Environment link --> [hopla-ucp-manager](https://ec2-34-251-95-192.eu-west-1.compute.amazonaws.com:8443)

~~~
demo-ucp-manager
63.32.101.223 demo-ucp-manager
~~~
Environment link --> [demo-ucp-manager](https://ec2-63-32-101-223.eu-west-1.compute.amazonaws.com:8443)

---
>NOTES:
> - Public IP info alternatives:
>   - curl -sS  ipinfo.io/ip
>   - curl -sS ifconfig.co
>   - wget http://ipinfo.io/ip -qO -
> - Bundle for admin user at /home/ubuntu/tools/bundle-admin
> ~~~
> docker run --rm \
> -v $(pwd):/OUTDIR hopla/ucptools \
> -u admin -p _YOUR-PASSWORD_ \
> -n https://_EXTERNAL-FQDN_:_PORT_ \
> -i _MY-USERID_
> ~~~ 

---