# playwright压测命令

```shell
nohup docker run --rm   --add-host docs.cncicore.com:192.168.0.201   -v /root/playwright_test:/work   -w /work   private-doc-registry.cn-qingdao.cr.aliyuncs.com/prometheus/my-playwright:1.0   python -u test-upload50m_youhua.py > 50m_20_10.log 2>&1 &
```
```shell
nohup docker run --rm   --add-host docs.cncicore.com:192.168.0.201   -v /:/work   -w /work   private-doc-registry.cn-qingdao.cr.aliyuncs.com/prometheus/my-playwright:1.0   python -u desktop-bak.py > ~/desktop.log 2>&1 &
```