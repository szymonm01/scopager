# Scopager
PHP package for managing class scopes (such as modules, packages, functionalities etc) and dependencies between them in modularized architecture

## Introduction and assumptions
Main goal of that package is to better manage scopes of classes, by packing them into several types of "wrappers", such as module, functionality, package.
This will help to control dependencies between them, and inform if you try to use some class outside its scope, what potentially could be architecture violation.

Base concepts are taken from Deptrac nad PhpDocumentor, but this package is combining them and is great tool to run on CI process, and for visualising dependencies inside your code.

## Wrappers
Main concept is to wrap all classes into functionalities, and functionalities into modules.
Definitions:

| Level | Wrapper | Description | Example usage |
| ----- | ----- | ----- | ---- |
| 0 | Shared | Code which could be used in every class in project. It should contains only required "tools" | EventDispacher, CommandBus |
| 1 | Module | Maximally independent structure, gathering functionalities of similar context | PaymentModule - module responsible of all actions assoscieted with payment service |
| 2 | Functionality | Code resposible of specific features/actions in its module context | CanclePayment - functionality responsible of cancelling pending payment |
| 3 | Block | Few classes performing very narrow task inside functionality | Classes responsible of sending email - prepare email to send, and for example call functionality from Mailing module |

Each class should define at least its Module. Other wrappers are not necessary, but it's recommended - if you declare them, you will have to split code into smaller pieces and increase readbility and cleanliness.

## Scopes
Each class should has defined scope. **If scope is not defined, it's Internal by default**.

### Predefined scopes
| Scope | Description | Example usage |
| ----- | ----- | ----- |
| Api | Classes in that scope can be used in all scopes of the same level, one level above and one level below | Api of Block can be used in Functionality, but not in Module. Api of Module and Shared can be used everywhere. |
| Internal | **Default scope**. Classes in that scope can be used only inside their wrapper and wrappers below | Internal class in Functionality can be used only in that specific Functionality and in all Blocks of that Functionality |

### Defining custom scopes
If you want to customize layers (scopes) in you application (for example to implement some architecture pattern), you can do it by creating `.scope.yml` file for any of specific wrapper (or by using wildcard `*`, and define it for any wrapper types or names).

File defining scopes should be placed in root directory of wrapper its describing, but in general it could be place anywhere in scanned code.

#### Example for new scopes in specific Module named `Cart`:
```yaml
# file: cart.scope.yml
wrappers:
  - type: Module
    name: Cart
    additional_layers:
      - name: ACL
        used_in: [Api, Internal, Repository]
        levels: [Module, Functionality, Block]
      - name: Repository
        used_in: [Internal]
        levels: [Functionality, Block]
```

#### Example for new scopes in all functionalities from specific module:
```yaml
# file: product.scope.yml
wrappers:
  - type: Functionality
    name: *
    parent_name: Product 
    additional_layers:
      - name: Service
        used_in: [Internal]
        levels: [Functionality, Block]
      - name: Entity
        used_in: [Service]
        levels: [Block]
```

## Throwing errors
### Using class outside its scope

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
Payment\MakeOrder\MakeOrderController:
line 8: "You are trying to use internal class from other functionality"
```

### Not defining class scope

```php
namespace Payment\CheckStatus;

#[Internal]
class StatusCheckingRepository
{
}
```

It this case error will be thrown:
```php
Payment\CheckStatus\StatusCheckingRepository:
line 3: "Scope of class is not defined"
```
