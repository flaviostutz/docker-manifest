# docker-manifest
This is a summary of best practices that we've been using when creating and operating Docker containers

### One Role per Container
   - a container is a “thing” to the person using it, not two or three things. Keep that way.
   - avoid commands per role, avoid two processes in the same container, avoid ENVs that change role

### Container as Decontamination Zone
   - clear UX pitfalls from original product, create a simple language

### Fight configuration entropy
   - reduce configuration options, aim clear useful deployment scenarios
   - mounting a config file as a volume is a great sin
   - ENV parameters must be documented and employ simple parameters. Use sed and other transformations at container startup to prepare the internal configuration according to the ENVs. Hide the mess from deployer.

### Deployable Readme
   - show at the top of the readme a minimal sample docker-compose.yml file that can be used to show your container running. Reference in compose file any other containers needed for the demo.
   - go straight and talk about how to get it running and how to see it doing what it is meant to do. don’t talk about the container itself

### Build starts in Buildfile
   - Your build must always be triggered by “docker build”. Avoid build params.
   - no Makefiles, scripts or other fancy schemes may be used to call docker build, but you can employ wherever you need inside the Dockerfile to get the job done.
   - always use multi-stage builds to hide your mess, like unit testing and so on

### Be Simple to enable Complexity
    - your container/process must be as simple as possible. A complex system will emerge from well made container units working together through their network interfaces
    - the more complex your container, the simpler the system you will manage to run sanely (or else get into a hole of madness with wires everywhere)

### Disposable like Trash
    - try as hard as you can to give the sensation that your container may be deleted at any time and another may be run and continue the job anywhere, even for Stateful scenarios (explain Ceph example)
    - your container instance should never be backed up or copied to anywhere. It is trash!

### Logs must be streamed only to the stdout/stderr
     - Probably the usage of logs will be performed outside your container. Leave the heavy lifting of storing, querying, removing, understanding and using to someone else, specialized on this matters.

### One Git Repo per Container
   - Dockerfile and docker-compose.yml used to build/dev test at project root
   - Any specific deployment of the container should not be in this repo, as this is a “build” repo. Create another repo for this task with the specific resources for your deployment.
   - schemes for creating various containers with a single project normally are forgetting to use Dockerfile appropriately and create ambiguities about which files are needed for which container role. Normally the thing is simpler than it seems when breaking in parts.
