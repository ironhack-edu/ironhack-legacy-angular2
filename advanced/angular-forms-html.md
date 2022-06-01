![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Angular | Forms HTML
Boilerplate code or boilerplate refers to sections of code that have to be included in many places with little or no alteration.
```html
<form (ngSubmit)="submitForm(form)" #form="ngForm" novalidate [ngClass]="{'is-processing': processing}">

  <div class="field" [ngClass]="{'has-error': feedbackEnabled && usernameField.errors}">
    <label>...</label>
    <input type="text" name="username" [(ngModel)]="username" #usernameField="ngModel" required minlength="6" [disabled]="processing"/>
    <div *ngIf="feedbackEnabled && usernameField.errors">
      <p *ngIf="usernameField.errors.required" class="validation">username required</p>
      <p *ngIf="usernameField.errors.minlength" class="validation">username is too short</p>
    </div>
  </div>

  <div class="field submit">
    <button type="submit" [disabled]="processing">Login</button>
    <div *ngIf="feedbackEnabled && form.invalid">
      <p class="validation">there are errors in the form, please review the fields</p>
    </div>
  </div>
</form>

```

