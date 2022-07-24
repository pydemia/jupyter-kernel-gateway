
<!-- https://hub.docker.com/r/elyra/nb2kg/

```bash
# docker run -t --rm \
#     -p 9002:9002 \
#     -p 9003:8888 \
#     -e GATEWAY_HOST=<gateway-hostname> \
#     -e NB_PORT=9002 \
#     -e KG_HTTP_USER=bob \
#     -v <host-notebook-directory>:/home/jovyan/work \
#     elyra/nb2kg

```

```bash
docker run -t --rm \
    -p 9002:9002 \
    -p 9003:8888 \
    -e GATEWAY_HOST=jkg \
    -e NB_PORT=9002 \
    -e KG_HTTP_USER=bob \
    -v `pwd`:/home/jovyan/work \
    elyra/nb2kg:2.4.0

```

---

# 

https://jupyter-kernel-gateway.readthedocs.io/en/latest/getting-started.html#running-using-a-docker-stacks-image

```dockerfile
# start from the jupyter image with R, Python, and Scala (Apache Toree) kernels pre-installed
FROM jupyter/all-spark-notebook

# install the kernel gateway
RUN pip install jupyter_kernel_gateway

# run kernel gateway on container start, not notebook server
EXPOSE 8888
CMD ["jupyter", "kernelgateway", "--KernelGatewayApp.ip=0.0.0.0", "--KernelGatewayApp.port=8888"]
```


```bash
docker run --rm -it bitnami/jupyterhub

``` -->


## `HTTP` mode

```bash
jupyter kernelgateway \
    --KernelGatewayApp.ip=0.0.0.0 \
    --KernelGatewayApp.port=9999 \
    --KernelGatewayApp.api=kernel_gateway.notebook_http \
    --KernelGatewayApp.seed_uri=./http_api_python.ipynb
    # --KernelGatewayApp.seed_uri=./http_api_r.ipynb

```

* /_api/spec/swagger.json
* 

```bash
$ curl -v localhost:9999/ping
*   Trying 127.0.0.1:9999...
* Connected to localhost (127.0.0.1) port 9999 (#0)
> GET /ping HTTP/1.1
> Host: localhost:9999
> User-Agent: curl/7.78.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: TornadoServer/6.1
< Content-Type: text/plain
< Date: Sun, 24 Jul 2022 15:24:37 GMT
< Etag: "cb24c04e8b86279d12dd1e225a8b73deaaba3fd6"
< Content-Length: 19
<
{"hello": "world"}
* Connection #0 to host localhost left intact
```

## `websocket` mode


```bash
# token <Unicode>
jupyter kernelgateway \
    --debug \
    --KernelGatewayApp.ip=0.0.0.0 \
    --KernelGatewayApp.port=9999 \
    --KernelGatewayApp.api=kernel_gateway.jupyter_websocket \
    --KernelGatewayApp.auth_token='test' \
    --JupyterWebsocketPersonality.list_kernels=True \
    --config=../jupyter_kernel_gateway_config.py
```

* GET /api/kernelspecs
* GET /api/kernels
* POST /api/kernels
* GET /api/kernels/{kernel_id}
* DELETE /api/kernels/{kernel_id}
* POST /api/kernels/{kernel_id}/interrupt
* POST /api/kernels/{kernel_id}/restart

REQUEST with Auth: `token`

```bash
curl 'http://localhost:9999/api/kernelspecs?token=test'
```

```bash
$ curl localhost:9999/api/kernels \
    -H 'Authorization: token test'

$ curl 'localhost:9999/api/kernels?token=test'
[]

$ curl -X POST 'localhost:9999/api/kernels?token=test' \
    -d '{
  "name": "kernel-gw-py310"
}'
{"id": "f883866a-d538-47ca-9b7a-76af54fe8216", "name": "kernel-gw-py310", "last_activity": "2022-07-24T16:10:56.174864Z", "execution_state": "starting", "connections": 0}

$ curl 'localhost:9999/api/kernels?token=test'
[{"id": "f883866a-d538-47ca-9b7a-76af54fe8216", "name": "kernel-gw-py310", "last_activity": "2022-07-24T16:10:56.174864Z", "execution_state": "starting", "connections": 0}]

$ curl 'localhost:9999/api/kernels/{f883866a-d538-47ca-9b7a-76af54fe8216}?token=test'
{"id": "f883866a-d538-47ca-9b7a-76af54fe8216", "name": "kernel-gw-py310", "last_activity": "2022-07-24T16:10:56.174864Z", "execution_state": "starting", "connections": 0}

$ curl -X POST 'localhost:9999/api/kernels/{f883866a-d538-47ca-9b7a-76af54fe8216}/interrupt?token=test'
```




---

## DEMO

```bash
git clone https://github.com/jupyter/kernel_gateway_demos
cd kernel_gateway_demos/python_client_example
docker-compose build
docker-compose up
```

```bash
cd kernel_gateway_demos/python_client_example
docker-compose run client \
    --times=1 \
    --kernel-id=9538352d-1c1e-4164-b86a-0e1f67722785 \
    --code='
print("Job: Run code with the given kernel: Success")
'
```