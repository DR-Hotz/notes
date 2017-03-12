## Yarn


### Generic Resource Management

	Design Decisions:

	1. Move all complexity to AM
	2. Do NOT trust AM(user code, hence low authority)
	3. Protect Resource Management from malicious AM

	Main component : Resource Manager(RM) and Node Manager(NM)

	Others : Scheduler(pluggable in RM), ApplicationManager(in RM)

	RM : arbitrator of all cluster resources

	NM : responsible for launching the applications’ containers, monitoring their resource usage, reporting the same to the ResourceManager

	Resource Format : <resource-name, priority, resource-requirement, number-of-containers>

	1. resource-name :　hostname, rackname where the resource is desired or *
	2. priority : intra-application priority
	3. resource-requirement : memory and CPU (currently supported)
	4. number-of-containers : limits the total number of containers per request.

	Container : merely the right to use a specified amount of resources on a specific machine in the cluster

	Local Cache :
	yarn.nodemanager.localizer.cache.cleanup.interval-ms

	yarn.nodemanager.localizer.cache.target-size-mb : only includes resources with PUBLIC and PRIVATE visibility

	yarn.nodemanager.localizer.client.thread-count

	yarn.nodemanager.localizer.fetch.thread-count


### Job-specific Execution and Monitoring

	Main component : Application Master(AM)

	AM : a user job life-cycle manager, responsible for negotiating appropriate resource containers from the Scheduler, working with the NodeManager(s) to execute and monitor the containers and their resource consumption.it's all communication with the ResourceManager and NodeManager is encoded using extensible network protocols

		1. Scale up : by localizing application fault tolerance.

		2. Openness : application framework-specific code isolation from RM and NM.


### Working Paradigm

	1. job submission, when it's valid, a container for AM is allocated;
	2. AM applies for resource through resource request to RM;
	3. RM generates a lease(token-based security mechanism) for the resource, and pass it to AM throught its heartbeat;
	4. AM presents the lease to NM, NM creates container for use;
	5. During job run, AM keeping updating job specific info with NM;
	6. Job done or be terminated, NM reclaims the containter resources.


### Resource Manager

	Scheduling:
		FIFO Scheduler

		Capacity Scheduler : Administrators can configure soft limits and optional hard limits on the capacity allocated to each queue. Each queue has strict ACLs (Access Control Lists) that control which users can submit applications to individual queues

		Fair Scheduler : all applications get, on average, an equal share of resources over time. every application belongs to a queue



### Node Manager

	container launch context (CLC) : includes a map of environment variables, dependencies stored in remotely accessible storage, security tokens, payloads for NodeManager services, and the command necessary to create the process
	The CLC provides resource requirements (memory/CPU), job files, security tokens, and other information needed to launch an ApplicationMaster on a node


### Application Master

	Once the AM is started (as a container), it will periodically send heartbeats to the ResourceManager to affirm its health and to update the record of its resource demands
