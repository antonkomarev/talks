---?image=https://user-images.githubusercontent.com/1849174/34500991-094a66da-f01e-11e7-9a6c-0480f1564338.png&size=80% auto

---

@title[Introduction]

Laravel Love let people express how they feel about the content.

There are many different implementations in modern applications:

- Github Reactions
- Facebook Reactions
- YouTube Likes

---

# Theory

---

@title[Domain 1/2]

## Domain

@ul

- `Reacter` — one who reacts.
- `Reacterable` — model which act as Reacter: Person, User, Organization, etc.
- `Reactant` — subject which could receive Reactions.
- `Reactable` — model which act as Reactant: Article, Comment, etc.
- `Reaction` — the response that reveals Reacter's feelings or attitude.

@ulend

---

@title[Domain 2/2]

## Domain

@ul

- `Reaction Type` — type of the emotional response: Like, Dislike, Love, Hate, etc.
- `Reaction Weight` — importance added by Reaction to the Reactable content.
- `Reaction Summary` — computed statistical values of Reactions related to Reactable entities.

@ulend

---

# Practice

---

## Installation

Inside your application pull in the package through Composer.

```sh
$ composer require cybercog/laravel-love
```

And run database migrations.

```sh
$ php artisan migrate
```

---

## Usage

In example we will use `User` model as actor to demonstrate easiest implementation,
but in reality it could be replaced with `Person`, `Organization`, `Bot` or
any other type of model which implements `Reacterable` contract.

---

### Prepare Reacterable Model

Implement `Reacterable` contract in model which will act as Reacter
and use `Reacterable` trait bundled out of the box.

```php
use Cog\Contracts\Love\Reacterable\Models\Reacterable as ReacterableContract;
use Cog\Laravel\Love\Reacterable\Models\Traits\Reacterable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements
    ReacterableContract
{
    use Reacterable;
}
```

---

### Prepare Reactable Model

Implement `Reactable` contract in model which will receive reactions
and use `Reactable` trait bundled out of the box.

```php
use Cog\Contracts\Love\Reactable\Models\Reactable as ReactableContract;
use Cog\Laravel\Love\Reactable\Models\Traits\Reactable;
use Illuminate\Database\Eloquent\Model;

class Comment extends Model implements
    ReactableContract
{
    use Reactable;
}
```

---

## Use cases

---

### User oriented actions

---

#### 1. Allow User to act as Reacter.

If `Reacterable` model don't has related `Reacter` model yet, you need to create it.


```php
$user->reacter()->create();
```

*Creation of the `Reacter` need to be done only once and usually done automatically on `Reacterable` model creation.*

---

#### 2. Start User to act as Reacter.

We need to get `Reacter` model related to `User` model.

```php
$reacter = $user->reacter()->first();
```

*Then you will be able to use all `Reacter` methods.*

---

#### 3. Reacter reacts to Comment.

```php
$reacter->reactTo($comment->reactant, ReactionType::LIKE);
```

---

#### 4. Reacter wants to remove reaction from Comment.

```php
$reacter->unreactTo($comment->reactant, ReactionType::LIKE);
```

---

#### 5. Get all Reactables reacted by Reacter.

```php
$reactables = [];
$reactions = $reacter->reactions()->latest('id')->get();
foreach ($reactions as $reaction) {
    $reactant = $reaction->reactant()->first();

    $reactables[] = $reactant->reactable()->first();
}
```

Or just use service class:

```php
$service = new ReacterService($reacter);
$reactables = $service->reactables();

// ordered by `id`
$reactables = $service->reactablesOrderedBy('id', 'DESC');
```

---

#### 6. Reactions which Reacter has made.

```php
$reacter->reactions()->get();
```

---

### Content oriented actions

---

#### 1. Allow Comment to act as Reactant.

If `Reactable` model don't has related `Reactant` model yet, you need to create it.

```php
$comment->reactant()->create();
```

*Creation of the `Reactant` need to be done only once and usually done automatically on `Reactable` model creation.*

---

#### 2. Make Comment to act as Reactant.

```php
$reactant = $comment->reactant()->first();
```

---

#### 3. See who's reacted to Reactant.

```php
$reacterables = [];
$reactions = $reactant->reactions()->latest('id')->get();
foreach ($reactions as $reaction) {
    $reacter = $reaction->reacter()->first();

    $reacterables[] = $reacter->reacterable()->first();
}
```

Or just use service class:

```php
$service = new ReactantService($reactant);
$reacterables = $service->reacterables();

// ordered by `id`
$reacterables = $service->reacterablesOrderedBy('id', 'DESC');
```

---

#### 4. Reactions which Reactant has received.

```php
$reactant->reactions()->get();
```

---

#### 5. Get reactions summary of the Reactant.

```php
$reactant->reactionsSummary()->first();
```

Reaction Summary will include collection of objects with type and aggregated count.

```json
[
    {
        "type": "Like",
        "count": 4
    },
    {
        "type": "Dislike",
        "count": 8
    }
]
```

---

#### 6. Order Reactable entities by overall reactions count.

There are situations when even negative reactions should be regarded as content popularity.
Then 3 likes and 5 dislikes will produce reactions weight equals to 8.

```php
$comments = Comment::orderByReactionsCount('DESC');
```

---

#### 7. Order Reactable entities by exact reaction type count.

```php
$comments = Comment::orderByReactionsCountOfType('Like', 'DESC');
```

---

#### 8. Order Reactable entities by reaction weight.

When you want to sort content reactions by difference between Likes and Dislikes.
For example each Reaction of type Like weight equals to +1 when Dislike weigth equals to -1.
Then 3 likes and 5 dislikes will produce reaction total weight equals to -2.

```php
$comments = Comment::orderByReactionsWeight('DESC');
```

---

## Configuration

---

> TODO: Mutually exclusive reactions.
