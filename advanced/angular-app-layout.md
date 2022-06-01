![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Angular | App Layout
Boilerplate code or boilerplate refers to sections of code that have to be included in many places with little or no alteration.
```html
<header id="site-header">
  <div class="container">

    <h1 class="site-title"><a [routerLink]="['/']"> ... </a></h1>

    <nav class="site-nav">
      ...
    </nav>

  </div>
</header>

<div id="site-main" class="container">
  <router-outlet></router-outlet>
</div>

<footer id="site-footer">
  <div class="container">
    nice footer
  </div>
</footer>
```

