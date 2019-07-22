---?image=https://user-images.githubusercontent.com/1849174/34500991-094a66da-f01e-11e7-9a6c-0480f1564338.png&size=80% auto

+++

@title[Introduction]

## Laravel Love v7

Laravel Love is emotional part of the application.

It let people express how they feel about the content. Make any model reactable in a minutes!

There are many different implementations in modern applications:

@ul

- Github Reactions
- Facebook Reactions
- YouTube Likes
- Reddit Votes
- Medium Claps

@ulend

---

# Theory

+++

## Diagram

![UML](https://user-images.githubusercontent.com/1849174/43216180-f15619b0-9046-11e8-8df7-482258ab3c0c.png)

+++

## Reaction

The response that reveals Reacter's feelings or attitude.

+++

## Reacter

One who reacts.

+++

## Reacterable

Model which can act as Reacter:

- User
- Person
- Organization
- Customer
- etc.

+++

## Reactant

Subject which could receive Reactions.

+++

## Reactable

Model which can act as Reactant:

- Article
- Comment
- etc.

+++

## Reaction Type

Type of the emotional response:

- Like
- Dislike
- Love
- Sad
- etc.

+++

## Reaction Weight

Importance added by Reaction to the Reactable content.

+++

## Reaction Total

Aggregated statistical values of total reactions count & their weight related to Reactant.

## Reaction Counter

Aggregated statistical values of ReactionTypes related to Reactant.

---

# Practice

+++

## Installation

Pull in the package through Composer to your application.

```sh
$ composer require cybercog/laravel-love
```

Run database migrations.

```sh
$ php artisan migrate
```

+++

## Usage

In example application:

- `User` model will act as `Reacter`.
- `Comment` model will act as `Reactant`.

+++

### Setup Reacterable Model

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

Create database migration with artisan setup command.

```sh
$ php artisan love:setup-reacterable --model="App\User" --nullable
```

Run migration.

```sh
$ php artisan migrate
```

+++

### Setup Reactable Model

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

Create database migration with artisan setup command.

```sh
$ php artisan love:setup-reactable --model="App\Comment" --nullable
```

Run migration.

```sh
$ php artisan migrate
```

+++

### Setup Reaction Types

+++

## User oriented use cases

+++

#### 1. Allow User to act as Reacter

`Reacterable` registers as `Reacter` only once. It will be done automatically on each successful create of the`Reacterable`.

If `Reacterable` model don't has related `Reacter` model yet you can register it manually.

```php
$user->registerAsLoveReacter();
```

+++

#### 2. User start to act as Reacter

Get `Reacter` facade from `Reacterable` model.

```php
$reacter = $user->viaLoveReacter();
```

*Then you will be able to use all `Reacter` facade methods.*

+++

#### 3. Reacter reacts to Reactant

```php
$reacter->reactTo($comment, 'Like');
```

+++

#### 4. Reacter remove reaction from Reactant

```php
$reacter->unreactTo($comment, 'Like');
```

+++

#### 5. Reactions which Reacter has made

```php
$reactions = $reacter->getReactions();
```

+++


#### 6. Get all Reactables reacted by Reacter

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
$reactables = $service->reactablesOrderedBy('id', 'desc');
```

+++

#### 7. Check if Reacter reacted to Reactant

```php
$hasReacted = $reacter->hasReactedTo($comment);

$hasNotReacted = $reacter->hasNotReactedTo($comment);
```

To check specific reaction type reaction pass it as second argument.

```php
$hasReacted = $reacter->hasReacted($comment, 'Like');

$hasNotReacted = $reacter->hasNotReacted($comment, 'Like');
```

+++

## Content oriented use cases

+++

#### 1. Allow Comment to act as Reactant

If `Reactable` model don't has related `Reactant` model yet, you need to create it.

```php
$comment->reactant()->create();

// $comment->registerAsReactant();
```

*Creation of the `Reactant` need to be done only once and usually done automatically on `Reactable` model creation.*

+++

#### 2. Make Comment to act as Reactant

```php
$reactant = $comment->getReactant();

// $reactant = Reactant::fromReactable($comment);
```

+++


#### 3. Reactions which Reactant has received

```php
$reactions = $reactant->getReactions();
```

+++

#### 4. See who's reacted to Reactant

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
$reacterables = $service->reacterablesOrderedBy('id', 'desc');
```

+++

#### 5. Reactions Summary

```php
$reactionsSummary = $reactant->getReactionsSummary();
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

#### 6. Order Reactables by total Reactions

Concept for social applications when even negative reaction increase popularity of the content.

```php
$comments = Comment::query()
    ->withReactionSummary()
    ->orderBy('reactions_total_count', 'desc');
```

👍 3 likes and 👎 5 dislikes will produce 8 reactions total count.

+++

#### 7. Order Reactables by exact Reaction Type count

```php
$reactionType = ReactionType::fromName('Like');

$comments = Comment::query()
    ->withReactionCounterOfType($reactionType)
    ->orderBy('reactions_count', 'desc');
```

+++

#### 8. Order Reactables by Reactions Weight

When you want to sort content reactions by difference between Likes and Dislikes.

```php
$comments = Comment::query()
    ->withReactionSummary()
    ->orderBy('reactions_total_weight', 'desc');
```

Default Like weight equals to +1 and Dislike weight equals to -1.

👍 3 likes and 👎 5 dislikes will produce -2 total reactions weight.

---

# Management

+++

### Mutually exclusive reactions

**TODO**

+++

### Terminal commands

- Recalculate likes counter
- Add new Reaction Types
- List Reaction Types table

---

# Thank you

- [Laravel Love](https://github.com/cybercog/laravel-love)
- [Anton Komarev](https://ell.folio.su)
