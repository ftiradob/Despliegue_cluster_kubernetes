# KUBERNETES


(EXPLICACION DE LO QUE VAMOS A CONSTRUIR Y DEL ESCENARIO)


Lo primero que tenemos que hacer es instalar docker en nuestras 3 máquinas:

~~~
vagrant@kubernetes:~$ sudo apt install docker.io


~~~

Ahora vamos a hacer la instalación de una serie de herramientas para realizar la gestión del cluster:

- kubeadm -> Herramienta para crear el cluster. 
- kubelet -> Herramienta que se ejecuta en todos los nodos y se dedica a iniciar los pods y contenedores.
- kubectl -> Herramienta para controlar el cluster.

~~~
vagrant@kubernetes:~$ sudo apt-get update && sudo apt-get install -y apt-transport-https curl gnupg2
vagrant@kubernetes:~$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
OK
vagrant@kubernetes:~$ cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
> deb https://apt.kubernetes.io/ kubernetes-xenial main
> EOF
deb https://apt.kubernetes.io/ kubernetes-xenial main
vagrant@kubernetes:~$ sudo apt-get update
vagrant@kubernetes:~$ sudo apt-get install -y kubelet kubeadm kubectl
~~~

Marcamos los paquetes para que se retengan y no puedan actualizarse o eliminarse de forma automática:

~~~
vagrant@kubernetes:~$ sudo apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
~~~
