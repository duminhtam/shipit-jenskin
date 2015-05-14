# Shipit with Jenskin (shipit-cli)
---
Shipit is a deployment tool, this document is talk about shipit-cli. Shipit-deploy is using shipit-cli and build an deployment flow

### Shipit-deploy Workflow tasks (never mind)
	- deploy
	  - deploy:init
	    - Emit event "deploy".
	  - deploy:fetch
	    - Create workspace.
	    - Initialize repository.
	    - Add remote.
	    - Fetch repository.
	    - Checkout commit-ish.
	    - Merge remote branch in local branch.
	    - Emit event "fetched".
	  - deploy:update
	    - Create and define release path.
	    - Remote copy project.
	    - Emit event "updated".
	  - deploy:publish
	    - Update symlink.
	    - Emit event "published".
	  - deploy:clean
	    - Remove old releases.
	    - Emit event "cleaned".
	- rollback
	  - rollback:init
	    - Define release path.
	    - Emit event "rollback".
	  - deploy:publish
	    - Update symlink.
	    - Emit event "published".
	  - deploy:clean
	    - Remove old releases.
	    - Emit event "cleaned".
	  - rollback:finish
	    - Emit event "rollbacked".
	- pending
	  - pending:log
	    - Log pending commits (diff between HEAD and currently deployed revision) to console.

Learning from `shipit-deploy` workflow we will self create the same workflow but work for **Jenskin build**

## Install
```bash
npm install shipit-cli
```
## Workspace
├── shipit-cli-demo
│   └── shipitfile.js
	
## Simple demo
```javascript
		module.exports = function (shipit) {
		//configuration
		  shipit.initConfig({
		    staging: {
		      servers: 'root@128.199.222.111'
		    }
		  });
		//create a task name pwd
		  shipit.task('pwd', function () {
			//run unix command pwd in remote server
		    return shipit.remote('pwd');
		  });
		  
		};
```

Run shipit from current workspace dir

```bash
shipit staging pwd
```

## Document
### Create your config for staging env
```javascript
 shipit.initConfig({
		    staging: {
		      servers: 'root@128.199.222.111',
		      my_a_config:'a_value',
		      my_b_config:'b_value'
		    }
		  });
```
#### Get your config
```javascript
	if(shipit.config.my_a_config == 'a_value'){
		//do your task
	}
```		 
### Create a simple task
```javascript
shipit.task('pwd', function () {
	//run unix command pwd in remote server
	return shipit.remote('pwd');
});
```
### Create a block task - Create a new Shipit task that will block other tasks during its execution
```javascript
shipit.blTask('pwd', function () {
  return shipit.remote('pwd');
});
```
#### Create a subsequence simple task
##### Using shipit.emit() to notify
```javascript
shipit.task('pwd', function () {	
	shipit.remote('pwd');
	//Notice that already run pwd cmd
	shipit.emit('pwded');

});

//What we do when pwded
shipit.on('pwded', function () {
	shipit.log('do your task when pwded');
});

```
##### Using shipit.start() to do a task
```javascript
shipit.task('pwd', function () {	
	shipit.remote('pwd');
	//Run a task
	shipit.start('task_after_pwd');

});

//What we do when pwded
shipit.task('task_after_pwd', function () {
	shipit.log('task task_after_pwd run');
});

```
### Run Unix command
#### On local
```
shipit.local('rm -rf /tmp/' + tmpDir);        
```
#### On remote env
```
shipit.remote('rm -rf /tmp/' + tmpDir);        
```
### Subsequence Unix command (local or remote)
```javascript
   shipit.local('ls').then(function(e){
   		//e.stdout response from unix command
    		if(e.stdout.trim() == 'shipitfile.js'){//ls some time has newline must trim()
    			console.log('ok found');
    		}
    }).then(function() {
    		//next
    });
```
### Remote copy from local to remote dest
```javascript
shipit.remoteCopy('/tmp/workspace', '/opt/web/myapp').then(...);
```
		  
