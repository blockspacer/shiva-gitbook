---
description: >-
  In this page you will find all the information you need on the ecs part of
  shiva, the api of the different classes, the game loop architecture.
---

# shiva::ecs

## How do the system works ?

### Systems

Shiva has 3 different kinds of systems:

* **PreUpdate**: These systems are the first to be updated in the game loop, they are generally used to retrieve user input, or manage network events for example.
* **LogicUpdate**: These systems are the second to be updated in the game loop, they are generally used for game logic such as movement or collisions for example.
* **PostUpdate**: These systems are the last to be updated in the game loop, they are generally used for rendering or interpolation for example.

The pseudo code look like this:

```typescript
function update()
{
  user defined;
}

function update_systems_type(system_type)
{
  for_each_systems;
  call update();
}

function update_systems() 
{
  call update_systems_type(pre_logic);
  while (frame_rate) {
    call update_systems_type(logic);
  }
  call update_systems_type(post_logic);
  return nb_systems_updated
}

GameLoop;
while (game_running) {
  call update_systems
}
return game_return_value;
```

{% hint style="info" %}
This game loop is based on the gafferon on games tutorial: [Fix your timestep](https://gafferongames.com/game-physics/fix-your-timestep).
{% endhint %}

### Diagram

![complete diagram](../../.gitbook/assets/code2flow_99b09.png)

![simple diagram](../../.gitbook/assets/code2flow_3c3ba.png)

## system\_type

### Description

This file contains an enum representing the different types of [systems presented previously](shiva-ecs.md#how-works-the-system), and three strong types, later used in template parameter by the [system class](shiva-ecs.md#system).

### system\_type API

{% tabs %}
{% tab title="Enum" %}
```cpp
enum system_type
{
  pre_update = 0,
  logic_update = 1,
  post_update = 2,
  size = 3,
};
```
{% endtab %}

{% tab title="Typedefs" %}
```cpp
using system_pre_update = NamedType<system_type,struct system_pre_update_tag>;
using system_post_update = NamedType<system_type, struct system_post_update_tag>;
using system_logic_update = NamedType<system_type, struct system_logic_update_tag>;
```
{% endtab %}
{% endtabs %}

## system\_manager

### Description

This class manage the **systems** of the entity component system. You are able to `add`, `remove`, `retrieve` , `update` or `delete` systems through it.

### Diagram

![system\_manager](../../.gitbook/assets/diagram.png)

### system\_manager API

{% tabs %}
{% tab title="Signature" %}
```cpp
class system_manager;
```
{% endtab %}

{% tab title="Constructor" %}
```cpp
explicit system_manager(entt::dispatcher &dispatcher,
                        entt::entity_registry &registry,
                        plugins_registry_t &plugins_registry) noexcept;
```

**Parameters**

* **entt::dispatcher** The `dispatcher` will be provided to the system when it is created.
* **entt::entity\_registry** The `entity_registry` will be provided to the system when it is created.
* ​[plugins\_registry](https://shiva.gitbook.io/project/~/edit/drafts/-LGoUJOpBxBYAeCO4Ght/shiva/modules/shiva-dll#plugins_registry-api) registry of the plugged systems
{% endtab %}

{% tab title="Functions" %}
| Function Name | Description |
| :--- | :--- |
| [update](shiva-ecs.md#update) | update the systems |
| [get\_system](shiva-ecs.md#get_system) | get a single system |
| [get\_systems](shiva-ecs.md#get_systems) | get multiple systems |
| [has\_system](shiva-ecs.md#has_system) | check if a system is present |
| [has\_systems](shiva-ecs.md#has_systems) | check if multiple systems are present |
| [mark\_system](shiva-ecs.md#mark_system) | mark a single system |
| [mark\_systems](shiva-ecs.md#mark_systems) | mark multiple systems |
| [enable\_system](shiva-ecs.md#enable_system) | enable a single system |
| [enable\_systems](shiva-ecs.md#enable_systems) | enable multiple systems |
| [disable\_system](shiva-ecs.md#disable_system) | disable a single system |
| [disable\_systems](shiva-ecs.md#disable_systems) | disable multiple systems |
| [create\_system](shiva-ecs.md#create_system) | create a single system |
| [load\_systems](shiva-ecs.md#load_systems) | create multiple systems |
| [nb\_systems](shiva-ecs.md#nb_systems) | get the number of systems |
| [load\_plugins](shiva-ecs.md#load_plugins) | load plugins and add it as systems |
| [get\_system\_by\_name](shiva-ecs.md#get_system_by_name) | get a single system by his name |
{% endtab %}
{% endtabs %}

#### update

```cpp
size_t update() noexcept
```

**Return value**

| Possible Name | Description |
| :--- | :--- |
| _**nb\_systems\_updated**_ | `size_t`  |

**Example**

```cpp
size_t nb_systems_updated = system_manager.update();
if (nb_systems_updated != 5) {
    /* Oh no, i was expected 5 systems to be executed in this game loop tick */
}
```

**Notes**

This is the function that updates your **systems**. Based on the logic of the different kinds of [shiva systems](shiva-ecs.md#how-works-the-system), this function take care of updating your systems in the right order.

{% hint style="info" %}
If you have not loaded any system into the **system\_manager** the function return 0.

If you decide to mark a system, it will be automatically **deleted** at the next loop tick through this function.
{% endhint %}

#### get\_system

```cpp
template <typename TSystem>
const TSystem &get_system() const noexcept;

template <typename TSystem>
TSystem &get_system() noexcept;
```

**Template parameters**

| Name | Description |
| :--- | :--- |
| _**TSystem**_ | `TSystem`  |

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**render\_system, log\_system**_ | `TSystem &const TSystem &` |

**Example**

```cpp
//! Non const version
auto& render_system = system_manager.get_system<game::render_system>();

//! const version
const auto& log_system = system_manager.get_system<game::log_system>();
```

#### get\_systems

```cpp
template <typename ...TSystems>
std::tuple<std::add_lvalue_reference_t<TSystems>...> get_systems() noexcept;

template <typename ...TSystems>
std::tuple<std::add_lvalue_reference_t<std::add_const_t<TSystems>>...> get_systems() const noexcept;
```

**Template parameters**

| Name | Description |
| :--- | :--- |
| _**TSystems**_ | `TSystems` |

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**tuple\_systems**_, \[_**system\_foo**_, _**system\_bar**_\] | `Tuple<TSystems &>Tuple<const TSystems &>` |

**Example**

```cpp
// Called from a const context
auto[system_foo, system_bar] = system_manager.get_systems<system_foo, system_bar>();

// Called from a non const context
auto[system_foo_nc, system_bar_nc] = system_manager.get_systems<system_foo, system_bar>();

// Get it as a tuple
auto tuple_systems = system_manager.get_systems<system_foo, system_bar>();
```

{% hint style="info" %}
This function recursively calls the [get\_system](shiva-ecs.md#get_systems) function
{% endhint %}

#### has\_system

```cpp
template <typename TSystem>
bool has_system() const noexcept;
```

**Template parameters**

| Name | Description |
| :--- | :--- |
| _**TSystem**_ | `TSystem` |

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**result**_ | `boolean` |

**Example**

```cpp
bool result = system_manager.has_system<my_game::render_system>();
if (!result) {
    //! Oh no, i don't have a rendering system.
}
```

#### has\_systems

```cpp
template <typename ... TSystems>
bool has_systems() const noexcept
```

**Template parameters**

| Name | Description |
| :--- | :--- |
| _**TSystems**_ | `TSystems` |

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**result**_ | `boolean` |

**Example**

```cpp
bool result = system_manager.has_systems<my_game::render_system,
                                         my_game::audio_system>();

if (!result) {
  //! Oh no, atleast one of the systems is not present
}
```

{% hint style="info" %}
This function recursively calls the [has\_system](shiva-ecs.md#has_system) function
{% endhint %}

#### mark\_system

```cpp
 template <typename TSystem>
 bool mark_system() noexcept
```

**Template parameters**

| Name | Description |
| :--- | :--- |
| _**TSystem**_ | `TSystem` |

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**result**_ | `boolean` |

**Example**

```cpp
bool result = system_manager.mark_system<my_game::render>();

if (!result) {
    //! Oh no the system has not been marked.
    //! Did you mark a system that is not present in the system_manager?
}
```

{% hint style="info" %}
This function marks a system that will be destroyed at the next tick of the game loop.
{% endhint %}

#### mark\_systems

```cpp
 template <typename ... TSystems>
 bool mark_systems() noexcept
```

**Template parameters**

| Name | Description |
| :--- | :--- |
| _**TSystems**_ | `TSystems` |

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**result**_ | `boolean` |

**Example**

```cpp
bool result = system_manager.mark_systems<my_game::render, my_game::audio>();

if (!result) {
    //! Oh no, atleast one of the system has not been marked.
}
```

{% hint style="info" %}
This function recursively calls the [mark\_system](shiva-ecs.md#mark_systems) function
{% endhint %}

#### enable\_system

```cpp
template <typename TSystem>
bool enable_system() noexcept
```

**Template parameters**

| Name | Description |
| :--- | :--- |
| _**TSystem**_ | `TSystem` |

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**result**_ | `boolean` |

**Example**

```cpp
bool result = system_manager.enable_system<my_game::render>();

if (!result) {
    //! Oh no, this system cannot be enabled.
    //! Did you enable a system that is not present in the system_manager?
}
```

{% hint style="info" %}
by default, a system is enabled.
{% endhint %}

#### **enable\_systems**

```cpp
template <typename ... TSystems>
bool enable_systems() noexcept
```

**Template parameters**

| Name | Description |
| :--- | :--- |
| _**TSystems**_ | `TSystems` |

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**result**_ | `boolean` |

```cpp
bool result = system_manager.enable_systems<my_game::render, my_game::audio>();

if (!result) {
    //! Oh no, atleast one of the requested systems cannot be enabled.
}
```

{% hint style="info" %}
This function recursively calls the [enable\_system](shiva-ecs.md#enable_system) function
{% endhint %}

#### disable\_system

```cpp
 template <typename TSystem>
 bool disable_system() noexcept
```

**Template parameters**

| Name | Description |
| :--- | :--- |
| _**TSystem**_ | `TSystem` |

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**result**_ | `boolean` |

**Example**

```cpp
bool result = system_manager.disable_system<my_game::render>();

if (!result) {
    //! Oh no the system_manager cannot disable this system.
}
```

{% hint style="info" %}
If you deactivate a system, it will not be destroyed but simply ignore during the game loop
{% endhint %}

#### disable\_systems

```cpp
 template <typename ... TSystems>
 bool disable_systems() noexcept
```

#### Template parameters

| Name | Description |
| :--- | :--- |
| _**TSystems**_ | `TSystems` |

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**result**_ | `boolean` |

**Example**

```cpp
bool result = system_manager.disable_systems<my_game::render, my_game::audio>();

if (!result) {
    //! Oh no, atleast one of the requested systems cannot be disabled.
}
```

{% hint style="info" %}
This function recursively calls the [disable\_system](shiva-ecs.md#disable_system) function
{% endhint %}

#### create\_system

```cpp
template <typename TSystem, typename ... TSystemArgs>
TSystem &create_system(TSystemArgs &&...args)
```

#### Template parameters

| Name | Description |
| :--- | :--- |
| _**TSystem**_ | `TSystem` |
| _**args**_ | `TSystemArgs` |

#### Return value

| Possible name | Description |
| :--- | :--- |
| _**render\_system**_ | `TSystem &` |

**Example**

```cpp
auto& render_system = system_manager.create_system<my_game::render>(/* args */);
```

#### load\_systems

```cpp
template <typename ...TSystems>
std::tuple<std::add_lvalue_reference_t<TSystems>...> load_systems() noexcept;
```

**Template parameters**

| Name | Description |
| :--- | :--- |
| _**TSystems**_ | `TSystems` |

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**tuple\_systems**_, \[_**system\_foo**_, _**system\_bar**_\] | `Tuple<TSystem &>` |

**Example**

```cpp
// Tuple packed
auto tuple_systems = system_manager.load_systems<system_foo, system_bar>();

// Tuple unpacked
auto [system_foo, system_bar] = system_manager.load_systems<system_foo, system_bar>();
```

#### nb\_systems

```cpp
size_t nb_systems() const noexcept;

size_t nb_systems(system_type sys_type) const noexcept;
```

#### Parameters

| Name | Description |
| :--- | :--- |
| _**sys\_type**_ | [`system_type`](shiva-ecs.md#system_type)\`\` |

#### Return value

| Possible name | Description |
| :--- | :--- |
| _**nb\_systems**_ | `size_t` |

**Example**

```cpp
//! Retrieve the number of systems.
size_t nb_systems = system_manager.nb_systems();

//! Retrieve the number of systems by a specific system_type.
size_t nb_systems_logic = system_manager.nb_systems(shiva::ecs::system_type::logic_update);
```

#### load\_plugins

```cpp
bool load_plugins() noexcept;
```

#### Return value

| Possible name | Description |
| :--- | :--- |
| _**result**_ | `boolean` |

**Example**

```cpp
bool result = system_manager.load_plugins();

if (!result) {
    // Oh no, atleast one of the plugins could not be loaded.
}
```

#### Notes

This function allow you to load the plugins of the plugins\_registry and create systems with the creator function of each plugins.

#### get\_system\_by\_name

```cpp
const base_system *get_system_by_name(std::string system_name,
                                      shiva::ecs::system_type type) const noexcept;
                                      
base_system *get_system_by_name(std::string system_name,
                                shiva::ecs::system_type type) noexcept;
```

#### Parameters

| Name | Description |
| :--- | :--- |
| _**system\_name**_ | `string` |
| _**type**_ | \`\`[`system_type`](shiva-ecs.md#system_type)\`\` |

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**render\_system**_ | `base_system *` |

**Example**

```cpp
base_system* render_system = system_manager.get_system_by_name("render_system", shiva::ecs::system_type::post_update);
if (render_system == nullptr) {
    //! Oh no, could not get the system properly.
}
```

**Notes**

* This function allow you to get a system by his name, used for get a specific plugin for example.

## base\_system

{% hint style="danger" %}
This class is an **abstract class**, it is documented but is present only to make type-erasure of the class system which is templated
{% endhint %}

{% hint style="info" %}
This class can be manipulated when using **plugins** to share data between them.
{% endhint %}

### Description

base class of shiva systems

### Diagram

![base\_system](../../.gitbook/assets/baseclass.png)

### base\_system API

{% tabs %}
{% tab title="Signature" %}
```cpp
class base_system;
```
{% endtab %}

{% tab title="Constructor" %}
```cpp
explicit base_system(entt::dispatcher &dispatcher,
                     entt::entity_registry &entity_registry,
                     const float &fixed_delta_time) noexcept
```
{% endtab %}

{% tab title="Functions" %}
| Function Name | Description |
| :--- | :--- |
| update | Pure virtual function, must be overriden by the client.  update the system. |
| get\_name | Pure virtual function, must be overriden by the client.  get the system name. |
| get\_system\_type\_RTTI | Pure virtual function, override by the system class. get the system type at runtime \(for plugins\).  |
| [mark](shiva-ecs.md#mark) | mark the system |
| [unmark](shiva-ecs.md#unmark) | unmark the system |
| [is\_marked](shiva-ecs.md#is_marked) | check if the system is marked. |
| [enable](shiva-ecs.md#enable) | enable the system |
| [disable](shiva-ecs.md#disable) | disable the system |
| [is\_enabled](shiva-ecs.md#is_enabled) | check if the system is enabled. |
| [im\_a\_plugin](shiva-ecs.md#im_a_plugin) | defines the system as a plugin. |
| [is\_a\_plugin](shiva-ecs.md#is_a_plugin) | check if the system is a plugin. |
| [get\_user\_data](shiva-ecs.md#get_user_data) | retrieve a user data previously set by set\_user\_data |
| [set\_user\_data](shiva-ecs.md#set_user_data) | set a user data for the system |
{% endtab %}
{% endtabs %}

#### mark

```cpp
void mark() noexcept
```

**Example**

```cpp
auto& render_system = system_manager.get_system<my_game::render>();
render_system.mark();
```

{% hint style="info" %}
This function marks the system, it will be destroyed in the next tick of the [game loop](shiva-ecs.md#diagram) by the [system\_manager](shiva-ecs.md#system_manager).
{% endhint %}

#### unmark

```cpp
void unmark() noexcept
```

**Example**

```cpp
auto& render_system = system_manager.get_system<my_game::render>();
render_system.unmark();
```

{% hint style="info" %}
This function unmark the system, allows the prevention of a destruction in the next tick of the [game loop](shiva-ecs.md#diagram) by the [system\_manager](shiva-ecs.md#system_manager).
{% endhint %}

#### is\_marked

```cpp
bool is_marked() const noexcept
```

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**result**_ | `boolean` |

**Example**

```cpp
auto& render_system = system_manager.get_system<my_game::render>();
bool result = render_system.is_marked();

if (!result) {
    //! render_system is not marked
}
```

#### enable

```cpp
void enable() noexcept
```

**Example**

```cpp
auto& render_system = system_manager.get_system<my_game::render>();
render_system.enable();
```

#### disable

```cpp
void disable() noexcept
```

**Example**

```cpp
auto& render_system = system_manager.get_system<my_game::render>();
render_system.disable();
```

{% hint style="info" %}
Take a look @ [disable\_system](shiva-ecs.md#disable_system)
{% endhint %}

#### is\_enabled

```cpp
bool is_enabled() const noexcept
```

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**result**_ | `boolean` |

**Example**

```cpp
auto& render_system = system_manager.get_system<my_game::render>();
bool result = render_system.is_enabled();

if (!result) {
    //! render_system is not enable
}
```

#### im\_a\_plugin

```cpp
void im_a_plugin() noexcept
```

**Example**

```cpp
auto& render_system = system_manager.get_system<my_game::render>();
render_system.im_a_plugin();
```

{% hint style="info" %}
This function defines the system as a plugin, and therefore use more feature in runtime to work properly

By default this function is called on plugins.
{% endhint %}

#### is\_a\_plugin

```cpp
bool is_a_plugin() const noexcept
```

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**result**_ | `boolean` |

**Example**

```cpp
auto& render_system = system_manager.get_system<my_game::render>();
bool result = render_system.is_a_plugin();

if (!result) {
    //! render_system is not a plugin
}
```

#### get\_user\_data

```cpp
void *get_user_data() noexcept;
```

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**data**_ | `void *` |

#### **Example**

```cpp
auto render_system = system_manager_.get_system_by_name("render_system", shiva::ecs::system_type::post_update);
auto input_system = system_manager_.get_system_by_name("input_system", shiva::ecs::system_type::pre_update);
input_system->set_user_data(render_system->get_user_data());
```

{% hint style="warning" %}
* This function retrieve a user data previously set by [set\_user\_data](shiva-ecs.md#set_user_data)
*  by default a user\_data is a void pointer equal to `nullptr`.
{% endhint %}

#### set\_user\_data

```cpp
void set_user_data(void *data) noexcept;
```

#### Parameters

| Name | Description |
| :--- | :--- |
| _**data**_ | `void *` |

**Example**

See the [example](shiva-ecs.md#example) above.

{% hint style="info" %}
* This function set a user data for this system
* This function is very useful to transfer \(with [get\_user\_data](shiva-ecs.md#get_user_data)\) data between plugins since they are base\_class.
* This function wcall on\_set\_user\_data callback at the epilogue, by default on\_set\_user\_data is empty and you need to override it if you need it.
{% endhint %}

{% hint style="warning" %}
* user should be aware here, that's manipulating void pointer is as your own risk.
{% endhint %}

## system

### Description

This class is the class that you have to inherit to create your systems

### Diagram

![system](../../.gitbook/assets/system.png)

### system API

{% tabs %}
{% tab title="Signature" %}
```cpp
template <typename TSystemDerived, typename TSystemType>
class system : public base_system;
```

**Template Parameters**

* **TSystemDerived** CRTP implementation of the system
* **TSystemType** Strong type representing the [system\_type](shiva-ecs.md#system_type) of the implemented system

{% hint style="success" %}
* This class is the class you will have to inherit to create your systems
{% endhint %}
{% endtab %}

{% tab title="Constructor" %}
```cpp
 template <typename ...Args>
 explicit system(Args &&...args) noexcept // *1
 
 system(shiva::entt::dispatcher &dispatcher,
               shiva::entt::entity_registry &entity_registry,
               const float &fixed_delta_time,
               std::string class_name) // *2
```

{% hint style="success" %}
\(1\) This constructor simply forward its arguments to base\_system.
{% endhint %}

{% hint style="info" %}
\(2\) This constructor is use for scripting.
{% endhint %}
{% endtab %}

{% tab title="Functions" %}
| Function Name | Description |
| :--- | :--- |
| update | Pure virtual function, must be overriden by the client. update the system. |
| [get\_system\_type](shiva-ecs.md#get_system_type) | get the system type at compile time |
| [get\_system\_type\_RTTI](shiva-ecs.md#get_system_type_rtti) | get the system type at runtime |
| [get\_name](shiva-ecs.md#get_name) | get the system name. |
{% endtab %}

{% tab title="Typedefs" %}
```cpp
template <typename TSystemDerived>
using logic_update_system = system<TSystemDerived, system_logic_update>;

template <typename TSystemDerived>
using pre_update_system = system<TSystemDerived, system_pre_update>;

template <typename TSystemDerived>
using post_update_system = system<TSystemDerived, system_post_update>;
```

#### Usage

```cpp
class system_implementation : public logic_update_system<system_implementation>
{
};
```
{% endtab %}
{% endtabs %}

#### get\_system\_type

```cpp
static constexpr system_type get_system_type() noexcept
```

**Return value**

| Possible  name | Description |
| :--- | :--- |
| _**sys\_type**_ | `system_type` |

#### **Example**

```cpp
auto sys_type = shiva::lua_system::get_system_type();

if (sys_type == shiva::ecs::system_type::logic_update) {
    //! Do things.
}

auto render_system = system_manager_.get_system_by_name("render_system", shiva::ecs::system_type::post_update);
sys_type = render_system.get_system_type_RTTI();

if (sys_type == shiva::ecs::system_type::post_update) {
    //! Do things.
}
```

#### get\_system\_type\_RTTI

```cpp
system_type get_system_type_RTTI() const noexcept final
```

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**sys\_type**_ | `system_type` |

**Example**

See the [example](shiva-ecs.md#example-1) above.

#### get\_name

```cpp
const std::string &get_name() const noexcept final
```

**Return value**

| Possible name | Description |
| :--- | :--- |
| _**sys\_name**_ | `string` |

**Example**

```cpp
auto& render_system = system_manager.get_system<my_game::render>();
const auto& sys_name = render_system.get_name();
assert(sys_name == "render_system");
```

