# delayed_job RIAK backend

## Installation

Add the gems to your Gemfile:

    gem 'delayed_job'
    gem 'delayed_job_riak'


## Riak
   
   Riak must be configured with allow_mul=true for this to work properly otherwise there is a real potential for multiple workers to claim the same job. 
   
   Riak works off of eventual consistency , so there is no data locking or anything of the sort.  Vector clocks track the updates made by clients and it is up to the clients to resolve the conflicted states that may arise. 
   
   If allow_mul is set to false it is somewhat close to last write wins and this can allow multiple workers to claim the same job. With allow_mul set to true riak will create siblings for conflicted objects (objects that do not descend from the same vector clock values).  This may happen when a worker claims a job and worker subsequently attempts to claim the same job. The first worker will be run the job and other worker will detect the conflict and ignore the job, the will not attempt to resolve the conflict though.  Conflict resolution will be taken care of by the worker that is running the job.
   
   When a worker runs a job after a successful completion the job is deleted from the queue.  If the job fails and needs to be rescheuled it will resolve any conflicts before that happens.  It does this by first copying the current attributes for the job and then reloading the job from riak.  If a conflict is detected the attributes are set to the values that have been copied and then the object is saved with the current vector clock , resolving the conflicted state.