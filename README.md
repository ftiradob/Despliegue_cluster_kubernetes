# KUBERNETES


(PONER VAGRANT Y EXPLICACION DE LO QUE VAMOS A CONSTRUIR Y DEL ESCENARIO)

En este escenario vamos a montar un cluster con Kubernetes usando 'kubeadm'


Lo primero que tenemos que hacer es instalar docker en las máquinas:

~~~
debian@kubeadm:~$ sudo apt install docker.io
debian@nodo1k8s:~$ sudo apt install docker.io
debian@nodo2k8s:~$ sudo apt install docker.io
~~~

Ahora vamos a hacer la instalación en las máquinas de una serie de herramientas para realizar la gestión del cluster:

- kubeadm -> Herramienta para crear el cluster. 
- kubelet -> Herramienta que se ejecuta en todos los nodos y se dedica a iniciar los pods y contenedores.
- kubectl -> Herramienta para controlar el cluster.

~~~
debian@kubeadm:~$ sudo apt-get update && sudo apt-get install -y apt-transport-https curl gnupg2
debian@kubeadm:~$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
OK
debian@kubeadm:~$ cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
deb https://apt.kubernetes.io/ kubernetes-xenial main
debian@kubeadm:~$ sudo apt-get update
debian@kubeadm:~$ sudo apt-get install -y kubelet kubeadm kubectl
~~~

Marcamos los paquetes para que se retengan y no puedan actualizarse o eliminarse de forma automática:

~~~
debian@kubeadm:~$ sudo apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
~~~

Apagamos la SWAP:

~~~
debian@kubeadm:~$ sudo swapoff -a
~~~

Finalmente comprobamos que se ha iniciado correctamente, y además vamos a almacenar el join y el token que nos aparece al final:

~~~
debian@kubeadm:~$ sudo kubeadm init --pod-network-cidr=192.168.20.0/24 --apiserver-cert-extra-sans=172.22.200.210
...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.0.13:6443 --token qt51tf.lvloa5cvg1xgxwd4 \
    --discovery-token-ca-cert-hash sha256:da68e3386649ce9a624b73abf6366b4f4aed3f3bd7b3efb28782867e0d45146e
~~~

Ahora vamos a seguir lo que nos indica el comando ejecutado con anterioridad:

~~~
debian@kubeadm:~$ mkdir -p $HOME/.kube
debian@kubeadm:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
debian@kubeadm:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
~~~

Tras esto vamos a realizar la instalación de 'Calico', que se trata de un proyecto que proporciona pod network a kubernetes para permitir la comunicación con otros nodos:

~~~
debian@kubeadm:~$ kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
~~~

Ahora vamos a conectar el nodo 1 con el master usando el token que adquirimos anteriormente:

~~~
debian@nodo1k8s:~$ sudo kubeadm join 10.0.0.13:6443 --token qt51tf.lvloa5cvg1xgxwd4 \
--discovery-token-ca-cert-hash sha256:da68e3386649ce9a624b73abf6366b4f4aed3f3bd7b3efb28782867e0d45146e
~~~

Hacemos lo mismo en el nodo 2:

~~~
debian@nodo2k8s:~$ sudo kubeadm join 10.0.0.13:6443 --token qt51tf.lvloa5cvg1xgxwd4 \
--discovery-token-ca-cert-hash sha256:da68e3386649ce9a624b73abf6366b4f4aed3f3bd7b3efb28782867e0d45146e
~~~

Finalmente nos dirigimos al nodo master y comprobamos que la unión se ha realizado con éxito:

~~~
debian@kubeadm:~$ kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
kubeadm    Ready    master   3h51m   v1.17.3
nodo1k8s   Ready    <none>   3h37m   v1.17.3
nodo2k8s   Ready    <none>   6m55s   v1.17.3
~~~

## INSTALACIÓN DE UNA APLICACIÓN EN UN CLUSTER CON KUBERNETES


