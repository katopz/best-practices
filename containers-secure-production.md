[Source](https://www.infoq.com/news/2016/12/containers-secure-production "Permalink to Running Docker Containers Securely in Production")

# Running Docker Containers Securely in Production

One way of hardening Docker containers in production is by making them immutable, i.e., read only. Other methods for running secure containers include minimizing the attack surface and applying both standard Linux hardening procedures as well as ones that are specific to a container environment.

A container [can be run in read-only mode][1] by passing the --read-only flag when starting it. This prevents any process from writing to the filesystem. Any attempt to write results in an error. Running such [immutable infrastructure][2] also ties in with other best practices for software deployment pipelines.

While immutability would prevent execution of any malicious scripts or changes that might happen via vulnerabilities present in other software running inside the container, how far is such a mode feasible for applications in the real world? For example, applications that generate log files and use databases would need writability.

One possible solution for logging would be to use a centralized logging system like Elasticsearch/Logstash/Kibana (ELK) so that all logs are collected in a central place, possibly another container, that is not directly accessed by the end user. Another alternative is to export the logs outside the container by using the --log-driver flag when starting the container. For applications that need write access to temporary directories like /tmp, a solution is to [mount a temporary file system][3] into the container for these directories.

Databases are not directly accessed by the end user, so the risk is lower. However, this does not preclude attacks unless the user-facing applications are hardened.

In cases where it is unavoidable to have a writable file system, Docker provides auditing and rolling back of changes. The file system in a Docker container is stacked as a series of layers. When a new container is created, a new layer is added on top which can be written to. The Docker storage driver hides this behind the scenes and presents it as a regular file system to the user. Writes made to a running container are made to this new layer. This is generally called Copy-On-Write (COW).

Configuration drift, or changes from the expected configuration, are easy to detect in a Docker container. The 'docker diff' command shows changes made to the filesystem - whether they be files added, removed or modified.

In addition to running a read-only container if possible, [other][4] [recommendations][5] for securing containers in production are:

* Running a minimal image like [Alpine Linux][6], which was designed with security in mind. The kernel is patched with an unofficial port of grsecurity. [Grsecurity][7] is a set of security enhancements to the Linux kernel which includes access control and elimination of memory corruption based vulnerabilities by minimizing the ways that a system can be attacked.
* Enforcing resource (CPU/RAM) limits to prevent DoS attacks.
* Configuring thread and process limits in the operating system.
* Applying standard Linux kernel hardening procedures like sysctl hardening.
* Running a single application per container. This is recommended because it reduces the attack surface, i.e., the amount of possible vulnerabilities for a given container is limited to those that might be present in the application on that container.

[1]: https://diogomonica.com/2016/11/19/increasing-attacker-cost-using-immutable-infrastructure/
[2]: https://www.infoq.com/immutable_infrastructure
[3]: http://www.projectatomic.io/blog/2015/12/making-docker-images-write-only-in-production/
