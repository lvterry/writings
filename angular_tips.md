# AngularJS Tips

To record the things I learned when using AngularJS to make prototypes.

### Parent scope

One requirement today is to show a list of employees to the user and allow the user to select one of them and perform some action. I have the list of employees data defined in the controller like this:
```javascript
$scope.employees = [{name: 'Terry', age: 30, title: 'UX'}, {name: 'Jason', age: 24, title: 'Front-end'}];
```
Then I use a `ng-repeat` in the HTML to render the employees.
```html
<tr ng-repeat="e in employees">
  <td><input type="radio" value="{{ e.name }}" name="employee"></td>
  <td>{{ e.name }}</td>
  <td>{{ e.age }}</td>
  <td>{{ e.title }}</td>
</tr>
```
I want to give `tr` a conditional CSS class when the radio button inside it is checked, as the following code.
```html
<tr ng-repeat="e in employees" ng-class="{ selected: selected }">
  <td><input type="radio" value="{{ e.name }}" name="employee" ng-model="selected"></td>
  <td>{{ e.name }}</td>
  <td>{{ e.age }}</td>
  <td>{{ e.title }}</td>
</tr>
```
But it does not work.

Checking a radio button will make the row highlighted but checking another radio button won't make the previous highlight go away.

I googled a bit but was struggling how to phrase the question. Suddenly, from my Chrome dev tools, I saw Angular created nested scopes for `ng-repeat` — each row has its own scope, modifying `selected` in one row won't affect the value of it in other rows. I needed the model `selected` on a higher level. Luckily, all these scopes have a common parent. So instead of saying `ng-model="selected"`, we'll say `ng-model="$parent.selected"`. The completed code looks like the following:
```html
<tr ng-repeat="e in employees" ng-class="{ selected: $parent.selected }">
  <td><input type="radio" value="{{ e.name }}" name="employee" ng-model="$parent.selected"></td>
  <td>{{ e.name }}</td>
  <td>{{ e.age }}</td>
  <td>{{ e.title }}</td>
</tr>
```

### ng-class

I have three classes for a button in my HTML. As follows.
```html
<a href="#" class="button button-primary button-small">Edit</a>
```
I want to add a ```button-disabled``` class to it under some conditions. I started with the following:
```html
<a href="#" class="button button-primary button-small" ng-class="{ 'button-disabled': isItemEditable }">Edit</a>
```
It does not work.

Then I realize a HTML element can only take one ```class``` attribute. I had to write the code in the following way:
```html
<a href="#" ng-class="{ 'button button-primary button-small': true, 'button-disabled': isItemEditable }">Edit</a>
```
Seems a bit counter-intuitive, isn't it?

### Focus a text input

When user clicks a button, I want to clear the text input field and put the cursor into it. I thought `ng-focus` is for this so I wrote the following code:
```html
<button ng-click="clearText()">Clear Text</button>
<input type="text" ng-model="url" ng-focus="shouldClearText">
```
```javascript
$scope.clearText = function() {
  $scope.url = '';
  $scope.shouldClearText = true;
}
```
Again, it does not work. `ng-focus` is used to execute the expression once the user puts her cursor into the text field. Googled a bit for solution. Turns out there's no easy way to do it. But people already came up with [solutions](http://www.emberex.com/programmatically-setting-focus-angularjs-way/) by using Angular custom directive.
