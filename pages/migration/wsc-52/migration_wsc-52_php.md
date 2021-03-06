---
title: Migrating from WSC 5.2 - PHP
sidebar: sidebar
permalink: migration_wsc-52_php.html
folder: migration/wsc-52
---

## Comments

The [`ICommentManager::isContentAuthor(Comment|CommentResponse): bool`](https://github.com/WoltLab/WCF/blob/aa96d34130d58c150a35ebd8936f09c830ccd685/wcfsetup/install/files/lib/system/comment/manager/ICommentManager.class.php#L151-L158) was added.
A default implementation that always returns `false` is available when inheriting from `AbstractCommentManager`.

It is strongly recommended to implement `isContentAuthor` within your custom comment manager.
An example implementation [can be found in `ArticleCommentManager`](https://github.com/WoltLab/WCF/blob/aa96d34130d58c150a35ebd8936f09c830ccd685/wcfsetup/install/files/lib/system/comment/manager/ArticleCommentManager.class.php#L213-L219).

## Email Activation

Starting with WoltLab Suite 5.3 the user activation status is independent of the email activation status.
A user can be activated even though their email address has not been confirmed, preventing emails being sent to these users.
Going forward the new `User::isEmailConfirmed()` method should be used to check whether sending automated emails to this user is acceptable.
If you need to check the user's activation status you should use the new method `User::pendingActivation()` instead of relying on `activationCode`.
To check, which type of activation is missing, you can use the new methods `User::requiresEmailActivation()` and `User::requiresAdminActivation()`.

## `*AddForm`

WoltLab Suite 5.3 provides a new framework to allow the administrator to easily edit newly created objects by adding an edit link to the success message.
To support this edit link two small changes are required within your `*AddForm`.

1. Update the template.

    Replace:
    ```smarty
    {include file='formError'}

    {if $success|isset}
        <p class="success">{lang}wcf.global.success.{$action}{/lang}</p>
    {/if}
    ```

    With:
    ```smarty
    {include file='formNotice'}
    ```

2. Expose `objectEditLink` to the template.

    Example (`$object` being the newly created object):
    ```php
    WCF::getTPL()->assign([
        'success' => true,
        'objectEditLink' => LinkHandler::getInstance()->getControllerLink(ObjectEditForm::class, ['id' => $object->objectID]),
    ]);
    ```

## `{jslang}`

Starting with WoltLab Suite 5.3 the `{jslang}` template plugin is available.
`{jslang}` works like `{lang}`, with the difference that the result is automatically encoded for use within a single quoted JavaScript string.

Before:

```smarty
require(['Language', /* … */], function(Language, /* … */) {
    Language.addObject({
        'app.foo.bar': '{lang}app.foo.bar{/lang}',
    });

    // …
});
```

After:

```smarty
require(['Language', /* … */], function(Language, /* … */) {
    Language.addObject({
        'app.foo.bar': '{jslang}app.foo.bar{/jslang}',
    });

    // …
});
```
