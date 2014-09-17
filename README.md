ngcourse-notes

update service.service

```javascript
angular.module('erg.server', [])

.value('API_BASE_URL', 'http://ngcourse.herokuapp.com')

.factory('server', function($http, API_BASE_URL) {
    var service = {};

    service.get = function(path) {
        return $http.get(API_BASE_URL + path)
            .then(function(response) {
                return response.data;
            });
    };

    service.post = function(path, data) {
        return $http.post(API_BASE_URL + path, data);
    }

    service.put = function(path, data) {
        return $http.put(API_BASE_URL + path, data);
    };

    return service;
});
```

update task-servie

```javascript
'use strict';

angular.module('erg.tasks', ['erg.server'])
    .factory('tasks', function(server) {
        var service = {};

        service.getTasks = function() {
            return server.get('/api/v1/tasks');
        };

        service.getTask = function(taskId) {
            return server.get('/api/v1/tasks/' + taskId).then(function(response) {
                var task = response[0];
                return task;
            });
        }

        service.addTask = function(task) {
            return server.post('/api/v1/tasks', task);
        }

        service.updateTask = function(task) {
            return server.put('/api/v1/tasks/' + task._id, task);
        }

        service.getMyTasks = function() {
            return service.getTasks()
                .then(function(tasks) {
                    return filterTasks(tasks, {
                        owner: user.username
                    });
                });
        };

        return service;
    });

```

task-add.html

```html

<h1>Task Add</h1>
Owner:
<input ng-model='newTask.task.owner' />
<br/>Description:
<input ng-model='newTask.task.description' />
<button ng-click='newTask.addTask(newTask.task)'>Save</button>

```
task-detail.html
```html
<h1>Task Details 2</h1>
Task id: {{taskDetails._id}}</br>
Owner:
<input ng-model='taskDetails.task.owner' />
<br/>Description:
<input ng-model='taskDetails.task.description' />
<button ng-click='taskDetails.updateTask(taskDetails.task)'>Save</button>

```

task details ctrl, task add ctrl. 
```javascript
'use strict';

angular.module('erg')

.controller('TaskDetailsCtrl', function($stateParams, $state, tasks) {

    var scope = this;
    scope._id = $stateParams._id;
    scope.task = {};

    scope.updateTask = function(task) {
        tasks.updateTask(task).then(function(response) {
            $state.go('tasks', {}, {
                reload: true
            });
        });
    }

    tasks.getTask(scope._id).then(function(task) {
        scope.task = task;
        return task;
    });

}).controller('TaskAddCtrl', function($state, tasks) {
    var scope = this;
    scope.addTask = function(task) {
        tasks.addTask(task).then(function(response) {
            $state.go('tasks', {}, {
                reload: true
            });
        });
    }

});
```
