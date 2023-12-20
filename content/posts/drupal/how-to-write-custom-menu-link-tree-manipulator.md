---
title: "How to write custom menu link tree manipulator"
date: 2023-12-20T08:06:25+06:00
description: How to write a custom menu link tree manipulator that will allow you to alter tree.
hero: images/drupal-menu-tree.png
tags: ["drupal", "menu", "entity", "custom"]
categories: ["Basic"]
---

In this post, I will show you how to write custom menu link tree manipulator.

In Drupal 9/10, we can write a custom menu link tree manipulator. For example, If you want to load the menu link tree sorted. The Drupal core provides the [\Drupal\Core\Menu\DefaultMenuLinkTreeManipulators::generateIndexAndSort](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Menu%21DefaultMenuLinkTreeManipulators.php/function/DefaultMenuLinkTreeManipulators%3A%3AgenerateIndexAndSort/8.1.x). This manipulator loads the sorted menu link tree as nested array but the keys of array also have the index.

> In this blog I am going to write a custom manipulator that will return the menu link tree sorted by menu link weight and maintain the array key as menu link plugin id.

First, we need to create a custom module. Let's name it `acme`. Then, we need to create a service class. Let's name it `AcmeMenuTreeManipulators`. The service class should extend the `DefaultMenuLinkTreeManipulators`.

## Step1: Create a custom Menu Link Manipulator class

The `AcmeMenuTreeManipulators` class should look like below, This should exits in `acme/src/Services/AcmeMenuTreeManipulators.php`.

```php
<?php

namespace Drupal\acme\Services;

use Drupal\Core\Menu\DefaultMenuLinkTreeManipulators;

/**
 * Provides a couple of menu link tree manipulators.
 *
 */
class AcmeMenuTreeManipulators extends DefaultMenuLinkTreeManipulators {

  /**
   * Generates a unique index and sorts by it.
   *
   * @param \Drupal\Core\Menu\MenuLinkTreeElement[] $tree
   *   The menu link tree to manipulate.
   *
   * @return \Drupal\Core\Menu\MenuLinkTreeElement[]
   *   The manipulated menu link tree.
   */
  public function sort(array $tree) {
    $new_tree = [];
    foreach ($tree as $key => $v) {
      if ($tree[$key]->subtree) {
        $tree[$key]->subtree = $this->sort($tree[$key]->subtree);
      }
      $instance = $tree[$key]->link;

      $new_tree[$instance->getPluginId()] = $tree[$key];
    }
    // Custom sort the tree by weight and use the plugin id as a key.
    uasort($new_tree, [$this, 'sortByMenuLinkWeight']);
    return $new_tree;
  }

  /**
   * Sorts by menu link weight.
   *
   * @param \Drupal\Core\Menu\MenuLinkTreeElement $a
   *   The menu link tree element.
   * @param \Drupal\Core\Menu\MenuLinkTreeElement $b
   *   The menu link tree element.
   *
   * @return int
   *   The sort order.
   */
  public function sortByMenuLinkWeight($a, $b) {
    if ($a->link->getWeight() == $b->link->getWeight()) {
      return 0;
    }
    return ($a->link->getWeight() < $b->link->getWeight()) ? -1 : 1;

  }

}
```

## Step 2: Create a service definition

Now, we need to create a service definition for the `AcmeMenuTreeManipulators` class. The service definition should look like below, This should exits in `acme/department_access.services.yml`.

```yaml
services:
  acme.menu_tree_manipulators:
        class: Drupal\acme\Services\AcmeMenuTreeManipulators
        arguments: ['@access_manager', '@current_user', '@entity_type.manager', '@menu_item_extras.menu_link_tree_handler']
```

## Step 3: Use custom menu link tree manipulator when loading the menu tree

```php
$params = new MenuTreeParameters();
$tree = $this->menuLinkTree->load($menu, $params);
$default_manipulators = [
  ['callable' => 'menu.default_tree_manipulators:checkAccess'],
  // ['callable' => 'menu.default_tree_manipulators:generateIndexAndSort'],
  ['callable' => 'acme.menu_tree_manipulators:sort'],
];
$manipulators = array_merge($default_manipulators, $manipulators);
$tree = $this->menuLinkTree->transform($tree, $manipulators);
//dpm($tree)
```

That's it. This is how you can implement the custom menu link tree manipulator.
