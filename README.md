# Scopager
PHP package for managing class scopes (such as modules, packages, functionalities etc) and dependencies between them

## Introduction and assumptions
Main goal of that package is to better manage scopes of classes, by packing them into several types of "wrappers", such as module, functionality, package.
This will help to control dependencies between them, and inform if you try to use some class outside its scope, what potentially could be architecture violation.

Base concepts are taken from Deptrac nad PhpDocumentor, but this package is combining them and is great tool to run on CI process, and for visualising dependencies inside your code.

## Basic usage
### Defining scope of class

```php
namespace Payment\CheckStatus;

#[Module(name: 'Payment')]
#[Functionality(name: 'CheckStatus')]
#[Internal]
class StatusCheckingRepository
{
}
```

```php
namespace Payment\MakeOrder;

#[Module(name: 'Payment')]
#[Functionality(name: 'MakeOrder')]
#[Api]
class MakeOrderController
{
  __construct(private \Payment\CheckStatus\StatusCheckingRepository $statusCheckingRepository){}
}
```

It this case error will be thrown:
```php
Payment\MakeOrderMakeOrderController:
line 8: "You are trying to use internal class from other functionality"
```
