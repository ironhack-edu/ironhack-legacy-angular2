<!-- ![](https://i.imgur.com/1QgrNNw.png)

# Irontrello | Angular Architecture -->

![IronTrello](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_fcdb0427f3f5d65df64de7df182a4e65.png)

## Learning Goals

After this lesson, you will be able to:

- Generate a new project with SASS and a customized prefix.
- Understand the Client App Architecture.
- Structure the IronTrello Client App.
- Consume the IronTrello API.

## Introduction

Let's do a quick recap about the features we are going to implement in IronTrello. The main features of our application will be the following:

- We are going to have just one board, where we will be able to create as many lists and cards as we want.
- We are going to have several lists.
- In each list, we are going to have as many cards as we want.
- We will have a drag and drop feature, that will allow us to sort and transfer cards between lists. This drag and drop feature will also work for lists.
- We will be able to edit lists.
- We will be able to edit cards.

Here you have a screenshot of how the drag'n'drop is in the original Trello. In the next lesson, we are going to cover in depth how to apply this feature to our project.

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_f879c36b25e0c95b5c641904f26c1664.png)

## Project structure

### Setup

First we are going to analyze the structure of our project. We have created the project with the `angular-cli`, and set it to use [SASS](http://sass-lang.com/) by default.

:::info
:bulb: If you want to create an Angular project with SASS in the future, you can run this modified version of the `ng new` command:

```
ng new <project-name> --style=scss
```
:::

This will automatically configure our application to compile our `scss` files into `css` files. Cool, huh? :)

### Setup

In our Angular project, we have the usual structure. We are going to take a look at our `app` folder of in `client/src/`:

```
app/
├── board/
│   |   ├── board.component.html
│   |   ├── board.component.scss
│   |   └── board.component.ts
├── card/
│   |   ├── modal/
│   |   │   ├── modal.component.html
│   |   │   ├── modal.component.scss
│   |   │   └── modal.component.ts
│   |   ├── card.component.html
│   |   ├── card.component.scss
│   |   ├── card.component.ts
│   |   └── card.model.ts
├── header/
│   |   ├── header.component.html
│   |   ├── header.component.scss
│   |   └── header.component.ts
├── list/
│   |   ├── placeholder/
│   |   │   ├── placeholder.component.html
│   |   │   ├── placeholder.component.scss
│   |   │   └── placeholder.component.ts
│   |   ├── list.component.html
│   |   ├── list.component.scss
│   |   ├── list.component.ts
│   |   └── list.model.ts
├── shared/
│   |   ├── interfaces/
│   |   │   ├── generic-response.interface.ts
│   |   │   ├── index.ts
│   |   │   └── sortable-item.interface.ts
│   |   ├── card.service.ts
│   |   ├── dragula.service.ts
│   |   ├── list.service.ts
│   |   └── toastr-options.ts
├── app.component.ts
└── app.module.ts
```

There is a lot to cover here. Let's analyze each folder's content to see why we have so many files and folders.

**`board/`, `card/`, `header/`, and `list/` folders**

We have one folder for each component that will compose our application. Our application will have a header and a board. Then, inside the board we will have all the lists, and inside each list we will have as many cards as we want. Inside these folders, we are going to find the common files for each component:

- `*.component.html`
- `*.component.scss`
- `*.component.ts`

### Extra Folders
#### `card/modal/`

Inside the `card`, we find another folder called `modal`. This modal is a component that will show the card edit form while we are editing a card information. Again, as each component folder, we will find three different files: html, scss, and typescript.

#### `list/placeholder/`

This component will also be used to edit a list. Instead of using a modal, the lists are edited with a placeholder. This is: when we click over a list name, it will appear an input where we will be able to change the list title.

#### `shared`

Last, but not least, we have the `shared` folder, where we have several things:

- **`card.service.ts`** - Integration with the `card` API methods.
- **`list.service.ts`** - Integration with the `list` API methods.
- **`dragula.service.ts`** - All the drag'n'drop methods.
- **`toastr-options.ts`** - Configures `toastr` (we will see this package in the next section).
- **`interfaces`**(folder) - Three different interfaces:
  - **`generic-response.interface.ts`** - Interface that will define what a response from our API looks like.
  - **`sortable-item.interface.ts`** - Common fields that all sortable elements will have. In this case, the elements are cards and lists, and both have an `_id`, a `title`, and a `position`.
  - **`index.ts`** - Helps us export all the interfaces we have defined.

Let's see which packages we are going to use to implement IronTrello.

### Packages

We are going to see the packages we haven't used before in our Angular projects. There will be 4:

- [Bootstrap 4](https://v4-alpha.getbootstrap.com/) to style the application. We are going to use the version 4.
- [ng-bootstrap](ng-bootstrap.github.io) will help us with the modals we are going to have in our application.
- [ng2-dragula](https://github.com/valor-software/ng2-dragula) is a package that we will use to drag and drop our lists and cards. We are going to cover this in the following lesson.
- [ng2-toastr](https://github.com/PointInside/ng2-toastr) is a JavaScript library used to show toast notifications.

We can install all the packages as usual, with the `npm install` command. For bootstrap 4 we need to install and add styles/scripts to the angular-cli.json file.

## Architecture overview

The following graph shows how we are going to organize our application:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_ff6843e5b16e90b6cbe7e36cf666e7a1.png)


As you can see, we will have an `AppComponent` that will be composed by two different components: `Header` and `Board`. Then, the `Board` component will have several nested `List` components. Each list will contain several `Card` components. Finally, each `Card` component will have its own `Modal`, that will be used to edit the cards information.

## Model implementation

The best approach is to start from the model, even in the client. In second place we'll implement the services for the API interaction. Then we'll write our components top-to-bottom to integrate all the things.

### List Model

We have already created the `List` model as it follows:

```typescript=6
export class List implements SortableItem {
  _id: string;
  title: string;
  position: number;
  cards: Array<Card> = [];

  constructor ({
    _id, title, position, cards
  }) {
    this._id = _id;
    this.title = title;
    this.position = position;
    this.cards = cards.map((card) => new Card(card));
  }

  update(list) {
    this.title = list.title;
    this.position = list.position;
  }

  addCard(card: Card): Array<Card> {
    this.cards.push(card);
    this.sortCards();
    return this.cards;
  }

  private sortCards() {
    this.cards = _.orderBy(this.cards, ['position', 'title']);
    return this.cards;
  }
}
```

:::success
:exclamation: Discuss each of the methods we've implemented here.
:::

Note how we are implementing a `SortableItem` interface. During the development of the project, we have seen that both `list` and `card` are sortable, and they have some fields in common. We decided to create the the interface to be sure that all the `SortableItems` that we could have in the future have these fields.

### Interface Sortable Resources

The `SortableItem` interface is very simple, as you can see:

```ts
export interface SortableItem {
    _id: string;
    title: string;
    position: number;
}
```

As you may know, this interface will enforce us to add this fields to any model that implements it.

### Card Model

Our `card` model is even simplier:

```typescript
import { SortableItem } from './../shared/interfaces';
import { List } from './../list/list.model';

export class Card implements SortableItem {
    _id: string;
    title: string;
    description: string;
    position: number;
    dueDate: Date;
    list: string;
    created_at: Date;
    updated_at: Date;

    constructor(rawObj) {
        Object.assign(this, rawObj);
    }

    setList(id) {
        this.list = id;
    }
}

```


### Interface common server response (advanced)

We have created a Generic Response Interface that will be used as an Observable while we are interacting with the server. With this interface we can handle better the response we are getting from the server.

```ts
import { List } from '../../list/list.model';
import { Card } from '../../card/card.model';

export interface IronTrelloGenericResponse {
    message: string;
    error?: string;
    list?: List;
    card?: Card;
}
```

The following step is to create the different services we are going to use to consume the API we created on the server side.

## Services for API communication

We are going to create two different services: one for the lists and another one for the cards.

### List service

The `list` service has the code for the CRUD operations on the List resource, and you can find it in the `shared/list.service.ts` file. This is the code of the service:

```typescript=13
@Injectable()
export class ListService {
  LIST_ROUTE = '/list';
  ENDPOINT: string;
  lists: Array<SortableItem> = [];

  constructor(
    @Inject('BASE_ENDPOINT') private BASE,
    @Inject('API_ENDPOINT') private API,
    private cardService: CardService,
    private http: Http
  ) {
    this.ENDPOINT = this.BASE + this.API;
  }

  get(): Observable<SortableItem[]> {
    return this.http.get(`${this.ENDPOINT}${this.LIST_ROUTE}/`)
      .map((res) => res.json())
      .map((res) => {
        for (const list of res) {
          this.lists.push(new List(list));
        }

        this.lists = this.sortItems(this.lists);
        return this.lists;
      })
      .catch((err) => Observable.throw(err));
  }

  private getNextPosition(): number {
    if (this.lists.length !== 0) {
      const pos = _.last(this.lists).position;
      return pos + 1000;
    } else {
      return 0;
    }
  }

  private sortItems(items: Array<SortableItem>): Array<SortableItem> {
    return _.orderBy(items, ['position', 'title']);
  }

  create(list): Observable<Array<SortableItem>> {
    list.position = this.getNextPosition();

    return this.http.post(`${this.ENDPOINT}${this.LIST_ROUTE}/`, list)
      .map((res) => res.json())
      .map((newList) => {
        this.lists.push(new List(newList));
        this.lists = this.sortItems(this.lists);
        return this.lists;
      })
      .catch((err) => {
        return Observable.throw(err.json());
      });
  }

  createCard(card, listId: string): Observable<Array<SortableItem>> {
    return this.cardService.create(card)
      .map((newCard: Card) => {
        const list = (_.find(this.lists, { _id: listId })) as List;
        return list.addCard(newCard);
      });
  }

  edit(list: SortableItem): Observable<IronTrelloGenericResponse> {
    return this.http.put(`${this.ENDPOINT}${this.LIST_ROUTE}/${list._id}`, list)
      .map((res) => res.json())
      .catch((err) => Observable.throw(err.json()));
  }

  remove(list: SortableItem): Observable<IronTrelloGenericResponse> {
    return this.http.delete(`${this.ENDPOINT}${this.LIST_ROUTE}/${list._id}`)
      .map((res) => res.json())
      .catch((err) => Observable.throw(err.json()));
  }

  shiftCard(sourceList, targetList, cardId): void {
    const sList = _.find(this.lists, { _id: sourceList }) as List;
    const tList = _.find(this.lists, { _id: targetList }) as List;
    const _index = _.findIndex(tList.cards, { _id: cardId }) as number;
    const _el = _.find(tList.cards, { _id: cardId }) as Card;

    if (_index !== -1) {
      if (_index === 0) {
        if (tList.cards.length > 1) {
          _el.position = tList.cards[1].position - 1000;
        } else {
          _el.position = 0;
        }
      } else {
        if (tList.cards[_index - 1] && tList.cards[_index + 1]) {
          _el.position = (tList.cards[_index - 1].position + tList.cards[_index + 1].position) / 2;
        } else {
          _el.position = tList.cards[_index - 1].position + 1000;
        }
      }

      // Update with the latest list id
      _el.setList(tList._id);

      if (sourceList === targetList) {
        const subscription = this.cardService.edit(_el).subscribe(
          (res) => console.log('Card position updated', res),
          (err) => console.log('Update card error', err)
        );
      } else {
        const subscription = this.cardService.transfer(_el, sourceList, targetList).subscribe(
          (res) => console.log('Card position updated', res),
          (err) => console.log('Update card error', err)
        );
      }
    }

    tList.cards = this.sortItems(tList.cards) as Card[];
  }

  shiftList(listId): void {
    const _index = _.findIndex(this.lists, { _id: listId });
    const _el = _.find(this.lists, { _id: listId }) as List;

    if (_index !== -1) {
      if (_index === 0) {
        if (this.lists.length > 1) {
          _el.position = this.lists[1].position - 1000;
        } else {
          _el.position = 0;
        }
      } else {
        if (this.lists[_index - 1] && this.lists[_index + 1]) {
          _el.position = (this.lists[_index - 1].position + this.lists[_index + 1].position) / 2;
        } else {
          _el.position = this.lists[_index - 1].position + 1000;
        }
      }

      const subscription = this.edit(_el).subscribe(
        (res) => console.log('Update list position', res),
        (err) => console.log('Update list error', err)
      );
    }
  }
}
```

:::success
Analyze with your class mates and your teacher the whole code to be sure you understand all the methods and functionalities that are in the service.
:::

### Card service

As in the previous section, here we can find all the code for CRUD operations with the `Card` API methods we created in the server side. Here is the whole code:

```typescript=10
@Injectable()
export class CardService {

  CARD_ROUTE = '/card';
  ENDPOINT: string;
  cards: Array<Card> = [];

  constructor(
    @Inject('BASE_ENDPOINT') private BASE,
    @Inject('API_ENDPOINT') private API,
    private http: Http
  ) {
    this.ENDPOINT = this.BASE + this.API;
  }

  create(card): Observable<Card> {
    return this.http.post(`${this.ENDPOINT}${this.CARD_ROUTE}/`, card)
      .map((res) => res.json())
      .map((newCard) => new Card(newCard))
      .catch((err) => Observable.throw(err.json()));
  }

  edit(card: Card) {
    return this.http.put(`${this.ENPOINT}${this.CARD_ROUTE}/${card._id}`, card)
      .map((res) => res.json().card)
      .map((_card) => {
        card = new Card(_card);
        return card;
      })
      .catch((err) => Observable.throw(err.json()));
  }

  transfer(card: Card, from, to) {
    const body = {
      card,
      from,
      to
    };

    return this.http.put(`${this.ENDPOINT}${this.CARD_ROUTE}/${card._id}/transfer`, body)
      .map((res) => res.json())
      .catch((err) => Observable.throw(err.json()));
  }

  remove(card: Card): Observable<IronTrelloGenericResponse> {
    return this.http.delete(`${this.ENDPOINT}${this.CARD_ROUTE}/${card._id}`)
      .map((res) => res.json())
      .catch((err) => Observable.throw(err.json()));
  }
}
```

:::success
This code is much simpler than the previous one, but you have to be sure you understand it completely.
:::

## Top level components

As a rule, think about components in terms of `input` (objects) and `output` (events). From there, implement the logic related. I'll write, if present, input/output first, because they represent the component's API.

### App

The `app.component` will have just two different things: the header and the board.

```htmlmixed
<ih-header></ih-header>
<ih-board></ih-board>
```

### Header

The original Trello has a login, and the header has some functionalities that we could implement here. Since there's no login implemented in the IronTrello, it just has some static HTML code showing the brand.

### Board

The `board` component will fetch the lists and iterate through the Array using an `*ngFor`.
Each iteration will create a `ih-list` component passing one list with property binding. The `ih-list` component will also let the user create a new list.

Here you have the lists iteration in the component:

```htmlmixed=10
<div id="list-wrapper-container" [dragula]='"lists"' [dragulaModel]="lists">
  <trello-list [id]="list._id" *ngFor="let list of lists; let i = index" [list]="list"
  (onListEdit)="onListEdit($event)" (onListRemove)="onListRemove($event)"></trello-list>
</div>
```

Take a look at the `board.component.html` file to check out the different sections in the HTML. We already have defined the PlaceHolder to add a new list, and the confirmation remove template.

As you can see, there are some methods attached in the component.

:::success
Check out the different methods and functionalities defined in the `board` component, by analyzing the `board.component.html` and `board.component.ts` files.
:::

## Inner (application) components

Once we have seen the `board` component, it's time to see the nested components we can find on it: lists and cards.

### List

The list will receive as input the `list` object to show, and will output both a remove action or an edit one, to notify the parent (board component).

```ts
@Input() list: List;
@Output() onListRemove = new EventEmitter<List>();
@Output() onListEdit = new EventEmitter<List>();
```

This component will also let the user create a new card:

**`list.component.html`**

```htmlmixed=23
<trello-placeholder (onSave)="addCard($event)" [placeholder]="'Add a card...'"></trello-placeholder>
```

**`list.component.ts`**

```typescript=40
addCard(title) {
  this.listService.createCard({
    title: title,
    position: this.getNewPosition(),
    list: this.list._id
  }, this.list._id).subscribe(
    (cards: Array<Card>) => {
    console.log('cards', cards);
  },
    (err) => console.log('Card add error')
  );
}
```

Note how we are using the `trello-placeholder` component to add a new List. You can find this component inside the `list` folder, in the `placeholder` folder.

### Card

The card will receive a `card` object as input and notify the delete action. The edit action will happen inside the modal component and the parent (in this case the list) doesn't need to be notified.

```ts
@Input() card: SortableItem;
@Output() onDelete = new EventEmitter<string>();
```

The `card` component is super simple:

**`card.component.html`**

```htmlmixed
<div class="card" (click)="openCardModal(card)">
  <p class="title"> {{ card.title }} </p>
  <p class="description"> {{ card.description }} </p>
  <small> position: {{ card.position }} </small>
</div>
```

**`card.component.ts`**

```typescript=23
openCardModal(card) {
  const modalInstance = this.modalService.open(ModalComponent);
  modalInstance.componentInstance.originalCard = card;

  modalInstance
    .result.then((reason) => {
      this.onDelete.emit(this.card._id);
    });
}
```

### Modal

To finish up with this lesson, we are going to see the modal. The modal component receives as input the card to edit.

```ts
@Input() originalCard: Card;
```

We'll clone the input to edit a different card than the original one, only if the user saves we'll update the original one.

```typescript=30
ngOnInit() {
  // Edit a clone of the original card
  this.card = _.clone(this.originalCard);

  if (this.card.dueDate) {
    this.dueDateDT = this.ngbDateParserFormatter.parse(this.card.dueDate.toString());
  }
}
```

```typescript=56
editCard(field) {
  this.cardService.edit(_.merge(this.originalCard, this.card))
    .subscribe(
      (response) => {
      this.editingDescription = false;
      this.editingDate = false;
      this.onSuccess(response.message)
    },
    (err) => this.onError(err.message)
  );
}
```

With the following code, we are going to generate a successful toastr that will show the "Yayy!" message when we edit or delete a card:

```typescript=76
onSuccess(message: string) {
  this.toastr.success(message, 'Yayy!');
}
```

## Summary

In this lesson we have covered a lot of things. We have analyzed the whole Angular application that we have created to consume the different methods we defined in our server.

We have seen how to organize our code and which is the correct workflow to decide how to create the logic of our application. We have also implemented some of the features we needed in our application.

## Extra Resources

- [Trello](https://www.trello.com/)
