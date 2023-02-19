# How to dispatch an event and process returned values

You need to run an event having one or more listeners and, in your main code, you wish to retrieve some values once listeners have done their job.

In our example below, we'll fire a `SampleEvent` class and his `SampleListener`. The idea is to initialize an `employee`.

File `app/Providers/EventServiceProvider.php`

```php
protected $listen = [
    SampleEvent::class => [
        SampleListener::class,
    ],
];
```

For our sample, your `routes/web.php` can looks like this:

```php
use App\Employee;
use App\Events\SampleEvent;

Route::get('/', function () {
    $employee = new Employee();

    SampleEvent::dispatch($employee);

    echo 'FIRSTNAME is ' . $employee->getFirstName() . PHP_EOL;
    echo 'NAME      is ' . $employee->getLastName()  . PHP_EOL;
    echo 'PSEUDO    is ' . $employee->getPseudo()  . PHP_EOL;
});
```

What we do is:

1. Create a new `employee` based on the `Employee` class,
2. Call our `SampleEvent` event and give our new `employee`,
3. Let the magic happens,
4. Display the employee's first and last name.

Here, by default, our employee.

## File app/Employee.php

This class will initialize our employee and provide setters and getters.

By default, our employee will be called `John Doe (cavo789)`.

```php
<?php

namespace App;

class Employee
{
    public function __construct(
        private string $firstname = 'John',
        private string $lastname  = 'Doe',
        private string $pseudo    = 'cavo789'
    ) {
    }

    public function getFirstName(): string
    {
        return $this->firstname;
    }

    public function setFirstName(string $firstname)
    {
        $this->firstname = $firstname;
        return $this;
    }

    public function getLastName(): string
    {
        return $this->lastname;
    }

    public function setLastName(string $lastname)
    {
        $this->lastname = $lastname;
        return $this;
    }

    public function getPseudo(): string
    {
        return $this->pseudo;
    }

    public function setPseudo(string $pseudo)
    {
        $this->pseudo = $pseudo;
        return $this;
    }
};
```

## File app/Events/SampleEvent.php

Our event will receive an employee and make it private.

Make three setters public to allow listeners to update the first and the last name. Also allow to initialize the pseudo.

```php
<?php

namespace App\Events;

use App\Employee;
use Illuminate\Foundation\Events\Dispatchable;

class SampleEvent
{
    use Dispatchable;

    public function __construct(private Employee $employee)
    {
    }

    public function setFirstName(string $firstname): self
    {
        $this->employee->setFirstName($firstname);
        return $this;
    }

    public function setLastName(string $lastname): self
    {
        $this->employee->setLastName($lastname);
        return $this;
    }

    public function setPseudo(string $pseudo): self
    {
        $this->employee->setPseudo($pseudo);
        return $this;
    }
}
```

## File app/Listeners/SampleListener.php

Our listener logic. `SampleListener` will receive the `SampleEvent` as parameter and, thus, has access to all his public methods. We'll here update the first and the lastname, we'll not update the pseudo.

```php
<?php

namespace App\Listeners;

use App\Events\SampleEvent;

class SampleListener
{
    public function handle(SampleEvent $event): void
    {
        $event->setFirstName('Georges')->setLastName('Washington');
    }
}
```

## The result

If we run `curl localhost` in the console, we'll get the output below showing us it has worked perfectly as expected.

```bash
FIRSTNAME is Georges
NAME      is Washington
PSEUDO    is cavo789
```

If we edit back the `app/Providers/EventServiceProvider.php` file and comment the listener like below illustrated, our code will still works

```php
protected $listen = [
    SampleEvent::class => [
        // SampleListener::class,
    ],
];
```

```bash
FIRSTNAME is John
NAME      is Doe
PSEUDO    is cavo789
```
