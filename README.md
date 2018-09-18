# docker-manifest
This is a summary of best practices that we've been using when creating and operating Docker containers

### One Role per Container

   - a container is a “thing” to the person using it, not two or three things. Keep that way.
   - avoid commands per role, avoid two processes in the same container, avoid ENVs that change role
   - **Example:** Have separate containers for "API Gateway" and "My Application". Even if both use NodeJS, keep them separated, because they have different roles, different resource needs, and may even evolve to use different languages in the future.

### Container as Decontamination Zone

   - clear UX pitfalls from original product, create a simple language
   - use ENV variables as your friend. Avoid complex relationships between them
   - **Example:** https://github.com/flaviostutz/ceph-osd. ENV OSD_EXT4_SUPPORT=true makes Ceph OSDs work with ext4 filesystems for testing purposes. In this case, I discovered that I needed to use "osd max object name len = 256" on ceph.conf to avoid names with more than 256 chars (imagine how I discovered that!). A regular user doesn't need to know that to make a simple demo on his own notebook. He only needs a OSD_EXT4_SUPPORT...

### Fight configuration entropy

   - reduce configuration options, aim clear useful deployment scenarios
   - mounting a config file as a volume is a great sin. You lose versioning and testing during *build*.
   - ENV parameters must be documented and employ simple parameters. Use sed and other transformations at container startup to prepare the internal configuration according to the ENVs. Hide the mess from deployer.
   - if you end up having too much ENV parameters with complex relationships, maybe the best is to have the container extended (FROM mycontainer) and use the file configurations directly by adding them inside your container. The configuration will be versioned in your repository and built into the container image.
   - Complex configurations need development and testing during *build* phase, not during *run* phase!
   - In case of a bug in configuration files during production, you can simply re-run the previous versioned container image that has a compatible software version + configuration file version embedded.
   - **Example:** https://github.com/flaviostutz/prometheus
       - This is a Prometheus container that accepts ENV STATIC_SCRAPE_TARGET=myapp@app.example.com to configure target /metrics servers
       - This way you don't have to mount an arbitrary/unversioned prometheus.yml during run
       - During startup we create automatically a prometheus.yml based on ENV contents. The advantage is that this process is repetitive, previsible and testable. This container doesn't support the configuration of Alert Rules (yet), for example - it was a decision.
       - If you need Prometheus features that are not present in this container (as Record Rules, for example), you won't use this container. In this case, it is recommended to extend prometheus container (Dockerfile FROM prom/prometheus:v2.4.0) and add configuration files during container build. See an example at https://github.com/flaviostutz/schelly-grafana
       - The magic is: you can run this Prometheus container using simple ```docker run``` commands with no "server" file configurations hanging on here and there.

### Deployable Readme

   - show at the top of the readme a minimal sample docker-compose.yml file that can be used to show your container running. Reference in compose file any other containers needed for the demo.
   - go straight and talk about how to get it running and how to see it doing what it is meant to do. don’t talk about the container itself
   - **Example:** https://github.com/flaviostutz/ceph-monitor

### Build starts in Buildfile

   - Your build must always be triggered by “docker build”. Avoid build params.
   - no Makefiles, scripts or other fancy schemes may be used to call docker build, but you can employ wherever you need inside the Dockerfile to get the job done.
   - always use multi-stage builds to hide your mess, like unit testing and so on
   - **Example:** https://github.com/flaviostutz/schelly 

### Be Simple to Enable Complexity

   - your container/process must be as simple as possible. A complex system will emerge from well made container units working together through their network interfaces
   - the more complex your container, the simpler the system you will manage to run sanely (or else get into a hole of madness with wires everywhere)
   - **Example:** https://github.com/kerberos-io/machinery - an opensource CFTV server
     - Each container instance supports only one camera
     - If you need to handle 1000 cameras, just run 1000 container instances. If processing is high, spread those containers among thousand of hosts.
     - If you want to show all camera images side by side, just point the feed of each camera live stream to each container address on your central UI.

### Disposable Like Trash

   - try as hard as you can to give the sensation that your container may be deleted at any time and another one may be launched and continue the job anywhere, on any server of your cluster, even for Stateful scenarios
   - all persistent data must be stored in container volumes. Your container instance should never be backed up or copied to anywhere. It is trash!
   - **Example:** https://github.com/flaviostutz/ceph-osd
     - During startup it checks for existing data on storage. If not present it bootstrap its contents, else reuses it
     - Ceph itself (not because of the container) is very sensitive to changes in daemon availability and changes its internal data accordingly, automatically. When you remove a storage node (OSD), for example, it distributes its reponsabilities to another nodes transparently.

### Logs = stdout/stderr

   - Probably the usage of logs will be performed outside your container, so don't keep it inside.
   - Leave the heavy lifting of storing, querying, expiring, consolidating and processing those huge amounts of data to someone else, specialized on these matters.
   - **Example:** We've tested the stack Docker Container -> JournalD -> FluentBit -> Kafka -> Graylog
     - If any Graylog, Kafka or FluentBit dies, the containers keeps running and catches up when comes online again
     - You can plug multiple log analysers on Kafka topics in order to process specific log insights in parallel with the great features of Graylog for log aggregation, searching and some structuring transformations

### One Git Repo per Container

   - Dockerfile and docker-compose.yml used to build/dev test at project root
   - Any specific deployment of the container should not be in this repo, as this is a “build” repo. Create another repo for this task with the specific resources for your deployment.
   - schemes for creating various containers with a single project normally are forgetting to use Dockerfile appropriately and create ambiguities about which files are needed for which container role. Normally the thing is simpler than it seems when breaking in parts.
   - **Example:** Schelly
     - https://github.com/flaviostutz/schelly - core Schelly container
     - https://github.com/flaviostutz/schelly-grafana - a Grafana container prepared with Dashboards for Schelly metrics
     - https://github.com/flaviostutz/schelly-restic - webhook bridge that receives calls from Schelly server for performing backups with Restic
     - https://github.com/flaviostutz/schelly-backy2 - webhook bridge that receives calls from Schelly server for performing backups with Backy2
     
   
