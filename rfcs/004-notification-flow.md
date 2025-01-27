- Start Date: 2021-08-21
- RFC PR: (leave this empty)

# Summary

Kanvas Apps can send notifications, can handle different types of notifications, and can send them via multiple channels. The downside is that users cannot configure which type of notification they want to receive, via which channels, and with what importance.

This proposal tries to address these 3 concerns and their implementation into the Kanvas Core.

# Detailed design 

[Database Diagram](https://dbdiagram.io/embed/612720576dc2bb6073bbee53)

**Entities:**
- **Users Notifications Settings:** Users settings per app of which notifications they want to receive and via which channels.
- **Notifications Types:** List of the current app amiable notification by system module.
- **Users Notifications Entity importance:** Specify the importance at which Users get their notifications based on the weight specified by notifications types.

Before the system can handle sending notifications based on settings and frequencies, we have to provide endpoints for allowing uses to manage their settings.

# API
## Notification Importance Endpoints

**List all notifications importance for the current user.**
- `GET - /v1/users/{id}/notifications_importance`
    ```
    {
        id: 3,
        entity_id : 100 {user_id},
        system_modules_id: 1 //based on the system module we will know the entity_namespace
        importance_id : 1 //importance reference for whats configure on this app
    }
    ```

**List the importance for the current entity id in the system module.**
- `GET - /v1/users/{id}/notifications_importance?q=(entity_id:{id},system_modules_id:{id})`
- `GET - /v1/users/{id}/notifications_importance/{entity_id}/system_module/{id}`
    ```
    {
        id: 3,
        entity_id : 100 {user_id},
        system_modules_id: 1 //based on the system module we will know the entity_namespace
        importance_id : 1 //importance reference for whats configure on this app
    }
    ```

- `POST - /v1/users/{id}/notifications_importance` : Create notification importance
    ```
    {
        entity_id : 100 {user_id},
        system_modules_id: 1 //based on the system module we will know the entity_namespace
        importance_id : 1 //importance reference for whats configure on this app
    }
    ```

- `PUT - /v1/notifications_importance/{entity_id}/system_module/{id}` : Update notification importance
    ```
    {
        entity_id : 100 {user_id},
        system_modules_id: 1 //based on the system module we will know the entity_namespace
        importance_id : 1 //importance reference for whats configure on this app
    }
    ```

## Notification Endpoints

- `GET - /v1/users/{id}/notifications` : List of the user notification settings (all notification available for the app)
    ```
    {
        name : 'Notification name',
        notifications_type_id : x
        is_enabled : 1,
        parent_id : 0, //list only the notifications that are main 
        channel : {
            'json',
            'email'
        }
    }
    ```

- `GET - /v1/users/{id}/notifications/{id}` : Get one notification and it children
    ```
    {
        notifications_type_id : x
        is_enabled : 1
        channel : {
            'json',
            'email'
        },
        related : [
            {
                name : 'Notification name',
                notifications_type_id : x
                notification_key : Canvas\Notifications\Signup
                is_enabled : 1
                channel : {
                    'json',
                    'email'
                }
            }
        ]
    }
    ```

- `PUT - /v1/users/{id}/notifications/{id}` : Get one notification
    ```
    {
        notifications_type_id : x
        is_enabled : 1
        channel : {
            'json',
            'email'
        }
    }
    ```

**Delete all notification settings for a user.**
- `DELETE - /v1/users/{id}/notifications`
## User Endpoints

- `GET - /v1/users/{id}` 
    ```
    {
        new_notification: 1 , //the user object will now return the total # of unread notification
        system_modules_id: 2 , //add the system module reference to this object so it easy for the frontend to handle importance
    }
    ```

# Examples 

In order to use this update we will have to provide 2 new Controllers on Kanvas Core and a Trait to use Notification importance on any entity within the developers App.

### UserNotificationSettingsController

```php
/**
 * Handle users notification Settings
 **/
class UsersNotificationSettingsController extends BaseController
{
    public function __construct()
    {
        $this->model = new NotificationsSettings();

        $this->model->users_id = $this->userData->getId();
        $this->model->apps_id = $this->app->getId();
        $this->additionalSearchFields = [
            ['apps_id', ':', $this->apps->getId()],
            ['users_id', ':', $this->userData->getId()],
            ['is_deleted', ':', 0],
        ];
    }

    public function index();
    public function getById();
}

```

### UserNotificationimportanceController

```php
/**
 * Handle users notification Settings
 **/
class UsersNotificationEntityimportanceController extends BaseController
{
    public function __construct()
    {
        $this->model = new UserNotificationEntityimportance();

        $this->model->users_id = $this->userData->getId();
        $this->model->apps_id = $this->app->getId();
        $this->additionalSearchFields = [
            ['apps_id', ':', $this->apps->getId()],
            ['users_id', ':', $this->userData->getId()],
            ['is_deleted', ':', 0],
        ];
    }

    public function index();

    /**
     * Verify that the user info belongs to the current logged in user
     */
    public function getById();
    public function getByEntity(string $entityId, int $systemModulesId);
}

```

# Motivation

Kanvas Core already provides notifications management, but we have to expand it and allow users to control which types of notifications they want to receive. We need to expand the system by adding this model.

# Tradeoffs

What potential tradeoffs are involved with this proposal?

- Adding `parent_id` to notifications will add a new layer of complexity.
- Should it live on the core or as a package?
# Alternatives

This will no longer be an issue in existing or future apps.

# Unresolved questions

Should this be part of the core or live as its own package?