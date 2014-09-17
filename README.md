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


updated state definition
```javascript
  .state('tasks.detail', {
            url: '/{_id:[0-9a-fA-F]{24}}',
            views: {
                'taskDetail@tasks': {
                    controller: 'TaskDetailsCtrl as taskDetails',
                    templateUrl: '/app/sections/task-list/task-details.html'
                }
            }

        }).state('tasks.add', {
            url: '/add',
            views: {

                'taskDetail@tasks': {
                    controller: 'TaskAddCtrl as newTask',
                    templateUrl: '/app/sections/task-list/task-add.html'
                }

            }
        })
```

updated index.html

```html
 <div ng-app="erg">

        <div ng-controller="MainCtrl as main">

            <div ng-hide="main.isAuthenticated" ng-controller="LoginFormCtrl as loginForm">
                Enter username:
                <input ng-model="loginForm.username" />
                <br/>Password:
                <input type="password" ng-model="loginForm.password" />
                <br/>
                <div ng-show="main.errorMessage">{{ main.errorMessage}}</div>
                username: {{loginForm.username}}
                <button ng-click="main.login(loginForm.username, loginForm.password)">Login</button>
            </div>

            <div ng-show="main.isAuthenticated">
                Hello, {{main.username}}!
            </div>

            <div ui-view></div>
        </div>

        <div ui-view="parent"></div>

    </div>
```


updated task list

```html
<h1>My Tasks2</h1>


<div ng-show="main.isAuthenticated" ng-controller="TaskListCtrl as taskList">
    {{main.username}}, you've got {{taskList.numberOfTasks}} tasks
    <br/>
    <button ui-sref='tasks.add'>Add task</button>
    <div ui-view="taskDetail"></div>
    <hr>Filter:
    <input ng-model="searchString">
    </br>
    <!-- tasks: {{taskList.tasks | json }} -->
    <table>
        <tr>
            <th>Owner</th>
            <th>Task description</th>
            <th></th>
        </tr>
        <tr ng-repeat="task in taskList.tasks | filter:searchString ">
            <td>{{task.owner}}</td>
            <td>{{task.description}}</td>
            <td><a ui-sref="tasks.detail({_id: task._id})">edit</a>
            </td>
        </tr>
    </table>

</div>

```
