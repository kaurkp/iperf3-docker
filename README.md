# iperf3-docker
This is a docker image with iperf3 compiled with sctp support

You can: 
- directly pull the image "sofianinho/iperf3:3.6-ubuntu18.04" from docker hub
```sh
docker pull sofianinho/iperf3:3.6-ubuntu18.04
```
- or rebuild yourself
```sh
# You can remove the --build-arg for the proxy if you don't have one (lucky you!)
docker build --no-cache --build-arg http_proxy=$http_proxy --build-arg https_proxy=$https_proxy -t iperf3:local https://github.com/sofianinho/iperf3-docker.git#master
```
- or set up your own version of iperf with the build-arg "VERSION"
```sh
# list of available versions here: https://downloads.es.net/pub/iperf/
docker build --no-cache --build-arg http_proxy=$http_proxy --build-arg https_proxy=$https_proxy --build-arg VERSION=3.5 -t iperf3:3.5 https://github.com/sofianinho/iperf3-docker.git#master
```
More info on the iperf3 binary inside:

```terminal
$ iperf3 --help
Usage: iperf3 [-s|-c host] [options]
       iperf3 [-h|--help] [-v|--version]

Server or Client:
  -p, --port      #         server port to listen on/connect to
  -f, --format   [kmgtKMGT] format to report: Kbits, Mbits, Gbits, Tbits
  -i, --interval  #         seconds between periodic throughput reports
  -F, --file name           xmit/recv the specified file
  -A, --affinity n/n,m      set CPU affinity
  -B, --bind      <host>    bind to the interface associated with the address <host>
  -V, --verbose             more detailed output
  -J, --json                output in JSON format
  --logfile f               send output to a log file
  --forceflush              force flushing output at every interval
  -d, --debug               emit debugging output
  -v, --version             show version information and quit
  -h, --help                show this message and quit
Server specific:
  -s, --server              run in server mode
  -D, --daemon              run the server as a daemon
  -I, --pidfile file        write PID file
  -1, --one-off             handle one client connection then exit
  --rsa-private-key-path    path to the RSA private key used to decrypt
                            authentication credentials
  --authorized-users-path   path to the configuration file containing user
                            credentials
Client specific:
  -c, --client    <host>    run in client mode, connecting to <host>
  --sctp                    use SCTP rather than TCP
  -X, --xbind <name>        bind SCTP association to links
  --nstreams      #         number of SCTP streams
  -u, --udp                 use UDP rather than TCP
  --connect-timeout #       timeout for control connection setup (ms)
  -b, --bitrate #[KMG][/#]  target bitrate in bits/sec (0 for unlimited)
                            (default 1 Mbit/sec for UDP, unlimited for TCP)
                            (optional slash and packet count for burst mode)
  --pacing-timer #[KMG]     set the timing for pacing, in microseconds (default 1000)
  --fq-rate #[KMG]          enable fair-queuing based socket pacing in
                            bits/sec (Linux only)
  -t, --time      #         time in seconds to transmit for (default 10 secs)
  -n, --bytes     #[KMG]    number of bytes to transmit (instead of -t)
  -k, --blockcount #[KMG]   number of blocks (packets) to transmit (instead of -t or -n)
  -l, --length    #[KMG]    length of buffer to read or write
                            (default 128 KB for TCP, dynamic or 1460 for UDP)
  --cport         <port>    bind to a specific client port (TCP and UDP, default: ephemeral port)
  -P, --parallel  #         number of parallel client streams to run
  -R, --reverse             run in reverse mode (server sends, client receives)
  -w, --window    #[KMG]    set window size / socket buffer size
  -C, --congestion <algo>   set TCP congestion control algorithm (Linux and FreeBSD only)
  -M, --set-mss   #         set TCP/SCTP maximum segment size (MTU - 40 bytes)
  -N, --no-delay            set TCP/SCTP no delay, disabling Nagle's Algorithm
  -4, --version4            only use IPv4
  -6, --version6            only use IPv6
  -S, --tos N               set the IP type of service, 0-255.
                            The usual prefixes for octal and hex can be used,
                            i.e. 52, 064 and 0x34 all specify the same value.
  --dscp N or --dscp val    set the IP dscp value, either 0-63 or symbolic.
                            Numeric values can be specified in decimal,
                            octal and hex (see --tos above).
  -L, --flowlabel N         set the IPv6 flow label (only supported on Linux)
  -Z, --zerocopy            use a 'zero copy' method of sending data
  -O, --omit N              omit the first n seconds
  -T, --title str           prefix every output line with this string
  --extra-data str          data string to include in client and server JSON
  --get-server-output       get results from server
  --udp-counters-64bit      use 64-bit counters in UDP test packets
  --repeating-payload       use repeating pattern in payload, instead of
                            randomized payload (like in iperf2)
  --username                username for authentication
  --rsa-public-key-path     path to the RSA public key used to encrypt
                            authentication credentials

[KMG] indicates options that support a K/M/G suffix for kilo-, mega-, or giga-

iperf3 homepage at: https://software.es.net/iperf/
Report bugs to:     https://github.com/esnet/iperf
```

And:
```terminal
$ iperf3 --version
iperf 3.6 (cJSON 1.5.2)
Linux 1d51f4438e82 4.15.0-32-generic #35~16.04.1-Ubuntu SMP Fri Aug 10 21:54:34 UTC 2018 x86_64
Optional features available: CPU affinity setting, IPv6 flow label, SCTP, TCP congestion algorithm setting, sendfile / zerocopy, socket pacing, authentication
```


## Using the sctp with docker
Support of sctp in Docker is rather recent at the time of writing (August 2018). The official integration came with Docker 18.03.0. If there is any kind of error related to sctp, it should mean that your docker daemon is not the right version (excluding wrong parameters or something like that).

### Server
```sh
docker run -t --rm --name iperf-server -p 5201:5201/tcp -p 5201:5201/udp -p 5201:5201/sctp sofianinho/iperf3:3.6-ubuntu18.04 -s
```

### Client
```sh
docker run -t --rm --link iperf-server sofianinho/iperf3:3.6-ubuntu18.04 -c  iperf-server -p 5201  --nstreams 4 --sctp --time=10
```

### Notes on sctp support with docker
Please read: "docker/2018-08-17-sctp-and-docker-18.06-ce.md" on my repo [TIL](https://github.com/sofianinho/TIL)


## Add Yaml Files to enable deployment on kubernetes

Once you have successfully set up a [Kubernetes](https://kubernetes.io) cluster on your machines you will be all set to explore k8s descriptors. Several K8s solutions exist which creates a competitive platform revolving around clustering, flexibility, language and any other specification making a kubernetes clustering system tempting to deploy. In the following some of the many existing quick and fast option to test kubernetes clusters are presented, interested readers can go through the following systems are considered: [Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/), [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/), [Rancherk3s](https://rancher.com/docs/k3s/latest/en/), [k3d](https://github.com/rancher/k3d) etc.

So Far, in the above sections we introduce a experimentation environment which permits us to perform the implementation.We can start working in the direction of adding a Yaml based descriptor file. In particular, the generation of kubernetes deployment description in this section below.


### Installation

#### Deployment
The deployment descritor files named `deployment`in our case we are goig to create two different Yaml based descriptors on for Server and another for Client.

`Source https://github.com/sofianinho/iperf3-docker/tree/master/deployment`  

##### Server
Creating a server file via `server-deployment.yaml`

Finally, apply the deployments.

```bash
kubectl apply -f server-deployment.yaml
```
Similarly, 
##### Client
Creating a server file via `client-daemonset.yaml`

Eventually, apply the deployment.

```bash
kubectl apply -f client-daemonset.yaml
```
##### Services
Creating a service file defined in i.e clusterIP, NodePort, LoadBalancer `services`

Apply the deployment.

```bash
kubectl apply -f iperf3-clusterip.yaml
```

### Sample output
         
```bash
deployment.apps/iperf3-server-deployment created
daemonset.apps/iperf3-clients created
service/iperf3-tcp created
```




