---?image=https://user-images.githubusercontent.com/1849174/34500991-094a66da-f01e-11e7-9a6c-0480f1564338.png&size=80% auto

+++

@title[Introduction]

## Laravel Love v6

Laravel Love is emotional part of the application.

It let people express how they feel about the content.

There are many different implementations in modern applications:

@ul

- Github Reactions
- Facebook Reactions
- YouTube Likes

@ulend

---

# Theory

+++

### Reaction

The response that reveals Reacter's feelings or attitude.

+++

### Reacter

One who reacts.

+++

### Reacterable

Model acting as Reacter:

- Person
- User
- Organization
- etc.

+++

### Reactant

Subject which could receive Reactions.

+++

### Reactable

Model acting as Reactant:

- Article
- Comment
- etc.

+++

### Reaction Type

Type of the emotional response:

- Like
- Dislike
- Love
- Hate
- etc.

+++

### Reaction Weight

Importance added by Reaction to the Reactable content.

+++

### Reaction Summary

Computed statistical values of Reactions related to Reactables.

---

# Practice

+++

## Installation

Inside your application pull in the package through Composer.

```sh
$ composer require cybercog/laravel-love
```

And run database migrations.

```sh
$ php artisan migrate
```

+++

## Usage

In demo application `User` model will act as `Reacter`.

In reality it could be replaced with `Person`, `Organization`, `Bot` or
any other type of model which implements `Reacterable` contract.

+++

### Prepare Reacterable Model

Implement `Reacterable` contract in model which will act as Reacter
by using `Reacterable` trait bundled out of the box.

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

+++

### Prepare Reactable Model

Implement `Reactable` contract in model which will receive reactions
by using `Reactable` trait bundled out of the box.

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

+++

## User oriented use cases

+++

#### 1. Allow User to act as Reacter

If `Reacterable` model don't has related `Reacter` model yet, you need to create it.


```php
$user->reacter()->create();
```

*Creation of the `Reacter` need to be done only once and usually done automatically on `Reacterable` model creation.*

+++

#### 2. User start to act as Reacter

We need to get `Reacter` model related to `User` model.

```php
$reacter = $user->reacter()->first();
```

*Then you will be able to use all `Reacter` methods.*

+++

#### 3. Reacter reacts to Reactant

```php
$reacter->reactTo($reactant, $reactionType);
```

+++

#### 4. Reacter remove reaction from Reactant

```php
$reacter->unreactTo($creactant, $reactionType);
```

+++

#### 5. Get all Reactables reacted by Reacter

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

+++

#### 6. Reactions which Reacter has made

```php
$reacter->reactions()->get();
```

+++

#### 7. Check if Reacter reacted to Reactant

```php
$isReacted = $reacter
    ->isReactedTo($reactant);

$isNotReacted = $reacter
    ->isNotReactedTo($reactant);
```

```php
$isReacted = $reacter
    ->isReactedWithTypeTo($reactant, $reactionType);

$isNotReacted = $reacter
    ->isNotReactedWithTypeTo($reactant, $reactionType);
```

+++

## Content oriented use cases

+++

#### 1. Allow Comment to act as Reactant

If `Reactable` model don't has related `Reactant` model yet, you need to create it.

```php
$comment->reactant()->create();
```

*Creation of the `Reactant` need to be done only once and usually done automatically on `Reactable` model creation.*

+++

#### 2. Make Comment to act as Reactant

```php
$reactant = $comment->reactant()->first();
```

+++

#### 3. See who's reacted to Reactant

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

+++

#### 4. Reactions which Reactant has received

```php
$reactant->reactions()->get();
```

+++

#### 5. Reactions Summary

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

+++

#### 6. Order Reactables by total reactions

There are situations when negative reactions should increase content popularity too.

```php
$comments = Comment::query()
    ->orderByReactionsCount('DESC');
```

3 likes and 5 dislikes will produce reactions total equals to 8.

+++

#### 7. Order Reactables by exact reaction type count

```php
$comments = Comment::query()
    ->orderByReactionsCountOfType('Like', 'DESC');
```

+++

#### 8. Order Reactables by reaction weight

When you want to sort content reactions by difference between Likes and Dislikes.

```php
$comments = Comment::query()
    ->orderByReactionsWeight('DESC');
```

When Like weight equals to +1 and Dislike weight equals to -1,
then 3 likes and 5 dislikes will produce reactions weight equals to -2.

+++

## Configuration

+++

### Mutually exclusive reactions

> TODO: Mutually exclusive reactions. 

---

# Thank you

- [Laravel Love](https://github.com/cybercog/laravel-love)
- [Anton Komarev](https://ell.folio.su)
