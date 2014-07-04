Angular Tips

# Parent scope

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

Check a radio button will make the row highlighted but checking another radio button won't make the previous highlight disappear, even the radio button inside that is unchecked already. I googled a bit but was struggling how to phrase the question. Suddenly, from my Chrome dev tools, I saw Angular created nested scopes for `ng-repeat` — each row has its own scope, modifying `selected` in one row won't affect the value of it in other rows. I needed the model `selected` on a higher level. Luckily, all these scopes have a common parent. So instead of saying `ng-model="selected"`, we'll say `ng-model="$parent.selected"`. The completed code looks like the following:
```html
<tr ng-repeat="e in employees" ng-class="{ selected: $parent.selected }">
  <td><input type="radio" value="{{ e.name }}" name="employee" ng-model="$parent.selected"></td>
  <td>{{ e.name }}</td>
  <td>{{ e.age }}</td>
  <td>{{ e.title }}</td>
</tr>
```
