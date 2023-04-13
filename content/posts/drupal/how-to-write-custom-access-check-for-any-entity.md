---
title: "How to write custom access check for any entity"
date: 2023-04-13T08:06:25+06:00
description: How to write custom access check for any entity, which can check more than user permission and role.
# menu:
#   sidebar:
#     name: How to restore tmux lost session
#     identifier: rich-content
#     parent: sub-category
#     weight: 10
hero: images/access-check.png
tags: ["drupal", "access", "entity", "custom"]
categories: ["Basic"]
---

In this post, I will show you how to write custom access check for any entity, which can check more than user permission and role.

In Drupal 8/9, we can write custom access check for any entity, which can check more than user permission and role. For example, we can check if the user is the author of the node or not. If the user is the author of the node, then we can allow the user to edit the node. Otherwise, we can deny the user to edit the node.

> In this blog I am going to go with a scenario assuming that we have a `field_department` taxonomy field on both user and node entity. We want to allow the user to edit the node if the user's department is the same as the node's department. Otherwise, we want to deny the user to edit the node. A node can only have one department and a user can have multiple departments.

First, we need to create a custom module. Let's name it `department_access`. Then, we need to create a service class. Let's name it `DepartmentAccessCheck`. The service class should implement `AccessInterface`. The `AccessInterface` has one method `access`. The `access` method is used to check if the user has access or not.

## Step1: Create a custom access check class

The `DepartmentAccessCheck` class should look like below, This should exits in `department_access/src/Access/DepartmentAccessCheck.php`.

```php
<?php

namespace Drupal\department_access\Access;

use Drupal\Core\Access\AccessResult;
use Drupal\Core\Routing\Access\AccessInterface;
use Drupal\Core\Session\AccountInterface;
use Drupal\node\Entity\Node;
use Drupal\user\Entity\User;

/**
 * Checks access for displaying configuration translation page.
 */
class DepartmentAccessCheck implements AccessInterface {

  /**
   * Checks access for the Department Access Check.
   *
   * @param \Drupal\Core\Session\AccountInterface $account
   *   Run access checks for this account.
   * @param \Drupal\node\Entity\Node $node
   *   The node object.
   *
   * @return \Drupal\Core\Access\AccessResultInterface
   *   The access result.
   */
  public function access(AccountInterface $account, Node $node) {
    // Load user object from $account.
    $account = User::load($account->id());
    // Check if the node and current logged in user both has field_department.
    if ($node->hasField('field_department') && $account->hasField('field_department')) {
      // Check if the department field values are same.
      $node_department = $node
        ->get('field_department')
        ->first()
        ->get('entity')
        ->getTarget()
        ->getValue()
        ->getName();
      $user_department_terms = $account
      ->get('field_department')
      ->referencedEntities();

      $user_departments = [];
      foreach ($user_department_terms as $key => $entity) {
        $user_departments[] = $entity->label();
      }

      if (in_array($node_department, $user_departments)) {
        return AccessResult::allowed();
      }
    }
    return AccessResult::forbidden();
  }

}
```

Now, we need to create a route subscriber class. The route subscriber class should implement `RouteSubscriberInterface`. The `RouteSubscriberInterface` has one method `alterRoutes`. The `alterRoutes` method is used to alter the routes. In this method, we can add our custom access check to the routes.

The `DepartmentAccessCheck` class should look like below, This should exits in `department_access/src/Routing/RouteSubscriber.php`.

## Step 2: Create a route subscriber class

```php
<?php

namespace Drupal\department_access\Routing;

use Drupal\Core\Routing\RouteSubscriberBase;
use Symfony\Component\Routing\RouteCollection;

/**
 * Listens to the dynamic route events.
 */

class RouteSubscriber extends RouteSubscriberBase {

  /**
   * {@inheritdoc}
   */
  protected function alterRoutes(RouteCollection $collection) {
    // Check if the route exists.
    if ($route = $collection->get('entity.node.edit_form')) {
      // Add our department access check to the route.
      $route->setRequirement('_department_access_check', 'Drupal\department_access\Access\DepartmentAccessCheck::access');
    }
  }

}

```

## Step 3: Create a service definition

Now, we need to create a service definition for the `DepartmentAccessCheck` class. The service definition should look like below, This should exits in `department_access/department_access.services.yml`.

```yaml
services:
  department_access.access_check:
    class: Drupal\department_access\Access\DepartmentAccessCheck
    tags:
      - { name: access_check, applies_to: '_department_access_check' }
```

That's it. Now, we can check if the user has access to edit the node or not. If the user has same department as the node, then the user can edit the node. Otherwise, the user can't edit the node.
