# kubeadm script
##  Controlplane
```
sudo -i
```
```
bash <(curl -s https://raw.githubusercontent.com/rizwan141/KUBERNETES/main/script/cluster-setup/master.sh)
```

##  worker
```
sudo -i
```
```
bash <(curl -s https://raw.githubusercontent.com/rizwan141/KUBERNETES/main/script/cluster-setup/worker.sh)
```

##  run the printed kubeadm-join-command from the master on the worker
