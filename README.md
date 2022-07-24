# jupyter-kernel-gateway

```bash
./create-condaenv.sh kernel-gw-py310
```

```bash
conda activate kernel-gw-py310
jupyter kernelgateway --generate-config
mv $HOME/.jupyter/jupyter_kernel_gateway_config.py ./
```