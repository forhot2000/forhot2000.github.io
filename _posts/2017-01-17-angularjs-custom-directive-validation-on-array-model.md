---
layout: post
title:  AngularJS Custom Directive Validation on Array Model
date:   2017-01-17 17:19:00 +0800
categories: angular
---

我们常常需要自定义一些输入控件，如选择用户控件，在angularjs中允许我们添加自定义directive，
我们用这种方式来实现自定义控件。

```js
// js
app.directive('user-select', function($q, $window) {
  function asyncOpenUserSelectDialog(users, opts) {
    var deferred = $q.defer();
    // here is a demo, replace it with your code
    $window.setTimeout(function() {
      var selectedUsers = [{name: 'bob'},{name: 'john'}];
      deferred.solve(selectedUsers);
    }, 100);
    return deferred.promise;
  }

  return {
    restrict: 'EAC',
    scope: {
      ngModel: '=',
      multi: '='
    }
    template: [
      '<ul><li ng-repeat="user in users track by $index">{{user.name}}</li></ul>',
      '<button ng-click="open()">select</button>',
      '<button ng-click="clear()">clear</button>'
    ].join(''),
    link: function(scope, elem, attrs) {
      scope.users = scope.$parent.$eval(attrs.ngModel);

      scope.open = function() {
        asyncOpenUserSelectDialog(scope.users, { multi: scope.multi }).then(function(selectedUsers) {
          scope.users = selectedUsers;
        });
      };

      scope.clear = function() {
        scope.users = [];
      }
    }
  };
});
```

```html
<!-- html -->
<div user-select ng-model="users"></div>
```

上面就可以简单的实现了我们需要的用户选择控件。

然而，我们经常需要判断选择的用户不允许为空，所以，我们来为它添加required validator，我们还是利用angular自带的
ngRequired directive来验证。

```js
// js
app.directive('user-select', function($q, $window) {
  function asyncOpenUserSelectDialog(users, opts) {
    var deferred = $q.defer();
    // here is a demo, replace it with your code
    $window.setTimeout(function() {
      var selectedUsers = [{name: 'bob'},{name: 'john'}];
      deferred.solve(selectedUsers);
    }, 100);
    return deferred.promise;
  }

  return {
    restrict: 'EAC',
    scope: {
      ngModel: '=',
      multi: '='
    }
    require: 'ngModel',
    template: [
      '<ul><li ng-repeat="user in users track by $index">{{user.name}}</li></ul>',
      '<button ng-click="open()">select</button>',
      '<button ng-click="clear()">clear</button>'
    ].join(''),
    link: function(scope, elem, attrs, ctrl) {
      scope.users = scope.$parent.$eval(attrs.ngModel);

      ctrl.$isEmpty = function(value) {
        return !value || value.length === 0;
      };

      scope.open = function() {
        var users = ctrl.$viewValue || angular.copy(ctrl.$modelValue);
        asyncOpenUserSelectDialog(users, { multi: scope.multi }).then(function(selectedUsers) {
          // don't set scope.users, use ctrl.$setViewValue to set viewValue,
          // it will trigger ctrl to run $parsers to validate input
          ctrl.$setViewValue(selectedUsers);
        });
      };

      scope.clear = function() {
        var users = ctrl.$viewValue || angular.copy(ctrl.$modelValue);
        if (users) {
          users.splice(0, users.length);
          // in angularjs v1.2, we use ctrl.$setViewValue to trigger validate, 
          // after angularjs v1.3, we can use ctrl.$validate() to trigger validate
          ctrl.$setViewValue(users);
        }
      }

      scope.$parent.$watch(attrs.ngModel, function(users) {
        scope.users = users;
      }, true);

      if (attrs.ngRequired) {
        var validator = function(value) {
          var valid = ctrl.$isEmpty(value);
          ctrl.$setValidaty('required', valid);
          return valid? value: undefined;
        }

        ctrl.$parsers.push(minValidator);
        ctrl.$formatters.push(minValidator);
      }
    }
  };
});
```

```html
<!-- html -->
<div user-select ng-model="users" ng-required></div>
```

<a href="https://codepen.io/forhot2000/pen/oBzyWL?editors=0010" target="_blank">Try in CodePen</a>
