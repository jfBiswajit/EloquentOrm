### Overview
**Source:** https://medium.com/@Mahmoud_Zalt/eloquent-relationships-cheat-sheet-5155498c209  
**Source:** https://gist.github.com/avataru/35c77721ec37b70df1d345b19ba0e2a6  
**Source:** https://laracasts.com/discuss/channels/eloquent/eloquent-sync-associate?page=1  

|                       |One to one<br>(`1-1`) |One to many<br>(`1-n`)           |Poly one to many<br>(`1x-n`)     | Many to many<br>(`n-n`)              |Poly many to many<br>(`nx-n`)         |
|-----------------------|:--------------------:|:-------------------------------:|:-------------------------------:|:------------------------------------:|:------------------------------------:|
|Number of models       |2 only                |2 only                           |3 and above                      |2 only                                |3 and above                           |
|Number of tables       |2 (1/model)           |2 (1/model)                      |3+ (1/model)                     |3 (1/model + pivot)                   |4+ (1/model + pivot)                  |
|Pivot table            |-                     |-                                |-                                |required                              |required                              |
|Model A relation       |`hasOne`              |`hasMany`                        |`morhpMany` (all)                |`belongsToMany`                       |`morphToMany` (all)                   |
|Model B relation       |`belongsTo`           |`belongsTo`                      |`morphTo`                        |`belongsToMany`                       |`morphToMany`                         |
|Set relation on Model A|`save(B)`             |`saveMany([B1, B2])`<br>`save(B)`|`saveMany([B1, B2])`<br>`save(B)`|`attach([B1, B2])`<br>`sync([B1, B2])`|`saveMany([B1, B2])`<br>`save(B)`     |
|Set relation on Model B|`associate(A)->save()`|`associate(A)->save()`           |`associate(A)->save()`           |`attach([A1, A2])`<br>`sync([A1, A2])`|`attach([A1, A2])`<br>`sync([A1, A2])`|
|Foreign key holder     |Model B table (`A_id`)|Model B table (`A_id`)           |Model B table (`A_id`, `A_type`) |Pivot table (`A_id`, `B_id`)          |Pivot table (`A_id`, `A_type`, `B_id`)|

## One to One Relationship
##### Demo details:
In this demo we have 2 models (**Owner** and **Car**), and 2 tables (**owners** and **cars**).
##### Business Rules:
The **Owner** can own one **Car**.
The **Car** can be owned by one **Owner**.
##### Relations Diagram:
![Diagram][one-to-one]
##### Relationship Details:
The **Cars** table should store the **Owner ID**.
##### Eloquent Models:
```php
class Owner
{
    public function car()
    {
       return $this->hasOne(Car::class);
    }
}
class Car
{
    public function owner()
    {
        return $this->belongsTo(Owner::class);
    }
}
```
##### Database Migrations:
```php
Schema::create('owners', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
});
Schema::create('cars', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
    $table->integer('owner_id')->unsigned()->index()->nullable();
    $table->foreign('owner_id')->references('id')->on('owners');
});
```
##### Store Records:
```php
// Create relation between Owner and Car.
$owner->car()->save($car);
// Create relation between Car and Owner.
$car->owner()->associate($owner)->save();
```
##### Retrieve Records:
```php
// Get Owner Car
$owner->car;
// Get Car Owner
$car->owner;
```

## One to Many Relationship
##### Demo details:
In this demo we have 2 models (**Thief** and **Car**), and 2 tables (**thieves** and **cars**).
##### Business Rules:
The **Thief** can steal many **Cars**.
The **Car** can be stolen by one **Thief**.
##### Relations Diagram:
![Diagram][one-to-many]
##### Relationship Details:
The **Cars** table should store the **Thief ID**.
##### Eloquent Models:
```php
class Thief
{
    public function cars()
    {
       return $this->hasMany(Car::class);
    }
}
class Car
{
    public function thief()
    {
        return $this->belongsTo(Thief::class);
    }
}
```
##### Database Migrations:
```php
Schema::create('thieves', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
});
Schema::create('cars', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
    $table->integer('thief_id')->unsigned()->index()->nullable();
    $table->foreign('thief_id')->references('id')->on('thieves');
});
```
##### Store Records:
```php
// Create relation between Thief and Car.
$thief->cars()->saveMany([
   $car1, 
   $car2,
]);
// Or use the save() function for single model.
$thief->cars()->save($car);
// Create relation between Car and Thief.
$car->thief()->associate($thief)->save();
```
##### Retrieve Records:
```php
// Get Thief Car
$thief->cars;
// Get Car Thief
$car->thief;
```

## Polymorphic One to Many Relationship
##### Demo details:
In this demo we have 2 models (**Driver** and **Car**), and 3 tables (**drivers**, **cars** and a pivot table named **car_driver**).
##### Business Rules:
The **Driver** can drive many **Cars**.
The **Car** can be driven by many **Drivers**.
##### Relations Diagram:
![Diagram][poly-one-to-many]
##### Relationship Details:
The **Pivot** table “car_driver” should store the **Driver ID** and the **Car ID**.
##### Eloquent Models:
```php
class Driver
{
    public function cars()
    {
       return $this->belongsToMany(Car::class);
    }
}
class Car
{
    public function drivers()
    {
        return $this->belongsToMany(Driver::class);
    }
}
```
##### Database Migrations:
```php
Schema::create('drivers', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
});
Schema::create('cars', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
});
Schema::create('car_driver', function (Blueprint $table) {
    $table->increments('id');
    $table->integer('car_id')->unsigned()->index();
    $table->foreign('car_id')->references('id')->on('cars')->onDelete('cascade');
    $table->integer('driver_id')->unsigned()->index();
    $table->foreign('driver_id')->references('id')->on('drivers')->onDelete('cascade');
});
```
##### Store Records:
```php
// Create relation between Driver and Car.
$driver->cars()->attach([
   $car1->id,
   $car2->id,
]);
// Or use the sync() function to prevent duplicated relations.
$driver->cars()->sync([
   $car1->id,
   $car2->id,
]);
// Create relation between Car and Driver.
$car->drivers()->attach([
   $driver1->id,
   $driver2->id,
]);
// Or use the sync() function to prevent duplicated relations.
$car->drivers()->sync([
   $driver1->id,
   $driver2->id,
]);
```
##### Retrieve Records:
```php
// Get Driver Car
$driver->cars
// Get Car Drivers
$car->drivers
```

## Many to Many Relationship
##### Demo details:
In this demo we have 3 models (**Man**, **Woman** and **Car**), and 3 tables (**men**, **women** and **cars**).
##### Business Rules:
The **Man** (buyer) can buy many **Cars**. 
The **Woman** (buyer) can buy many **Cars**.
The **Car** can be bought by one buyer (**Man** or **Woman**).
##### Relations Diagram:
![Diagram][many-to-many]
##### Relationship Details:
The **Car** table should store the **Buyer ID** and the **Buyer Type**. 
_“buyer” is a name given to a group of models (Man and Woman). And it’s not limited to two. The buyer type is the real name of the model._
##### Eloquent Models:
```php
class Man
{
    public function cars()
    {
        return $this->morphMany(Car::class, 'buyer');
    }
}
class Woman
{
    public function cars()
    {
        return $this->morphMany(Car::class, 'buyer');
    }
}
class Car
{
    public function buyer()
    {
        return $this->morphTo();
    }
}
```
##### Database Migrations:
```php
Schema::create('men', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
});
Schema::create('women', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
});
Schema::create('cars', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
$table->integer('buyer_id')->unsigned()->index()->nullable();
    $table->string('buyer_type')->nullable();
});
```
##### Store Records:
```php
// Create relation between buyer (Man/Woman) and Car.
$man->cars()->saveMany([
   $car1, 
   $car2,
]);
$woman->cars()->saveMany([
   $car1, 
   $car2,
]);
// Or use the save() function for single model.
$man->cars()->save($car);
$woman->cars()->save($car);
// Create relation between Car and buyer (Men/Women).
$car1->buyer()->associate($man)->save();
$car2->buyer()->associate($woman)->save();
```
##### Retrieve Records:
```php
// Get buyer (Man/Woman) Cars
$men->cars
$women->cars
// Get Car buyer (Man and Woman)
$car->buyer
```

## Polymorphic Many to Many Relationship
##### Demo details:
In this demo we have 3 models (**Valet**, **Owner** and **Car**), and 4 tables (**valets**, **owners**, **cars** and **drivers**).
##### Business Rules:
The **Valet** (driver) can drive many **Cars**. 
The **Owner** (driver) can drive many **Cars**.
The **Car** can be driven by many drivers (**Valet** or/and **Owner**).
##### Relations Diagram:
![Diagram][poly-many-to-many]
##### Relationship Details:
The **Pivot** table “drivers” should store the **Driver ID**, **Driver Type** and the **Car ID**.
_“driver” is a name given to a group of models (Valet and Owner). And it’s not limited to two. The driver type is the real name of the model._
##### Eloquent Models:
```php
class Valet
{
    public function cars()
    {
        return $this->morphToMany(Car::class, 'driver');
    }
}
class Owner
{
    public function cars()
    {
        return $this->morphToMany(Car::class, 'driver');
    }
}
class Car
{
    public function valets()
    {
        return $this->morphedByMany(Man::class, 'driver');
    }

    public function owners()
    {
        return $this->morphedByMany(Woman::class, 'driver');
    }
}
```
##### Database Migrations:
```php
Schema::create('valets', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
});
Schema::create('owners', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
});
Schema::create('drivers', function (Blueprint $table) {
    $table->increments('id');
    $table->integer('driver_id')->unsigned()->index();
    $table->string('driver_type');
    $table->integer('car_id')->unsigned()->index();
    $table->foreign('car_id')->references('id')->on('cars')->onDelete('cascade');
});
```
##### Store Records:
```php
// Create relation between driver (Valet/Owner) and Car.
$valet->cars()->saveMany([$car1, $car2]);
$owner->cars()->saveMany([$car1, $car2]);
// Or use the save() function for single model.
$valet->cars()->save($car1);
$owner->cars()->save($car1);
// Create relation between Car and driver (Valet/Owner).
$car->valets()->attach([
    $valet1->id,
    $valet2->id,
]);
$car->owners()->attach([
    $owner1->id,
    $owner2->id,
]);
// Or use the sync() function to prevent duplicated relations.
$car->valets()->sync([
    $valet1->id,
    $valet2->id,
]);
$car->owners()->sync([
    $owner1->id,
    $owner2->id,
]);
```
##### Retrieve Records:
```php
// Get driver (Valet/Owner) Cars
$valet->cars
$owner->cars
// Get Car drivers (Valet and Owner)
$car->owners
$car->valets
```

[one-to-one]: https://cdn-images-1.medium.com/max/1600/1*gFtdOGZWSzepjFh5ar7bMw.png
[one-to-many]: https://cdn-images-1.medium.com/max/1600/1*Cn4si4bSwFESO1zIjVT0FA.png
[many-to-many]: https://cdn-images-1.medium.com/max/1600/1*L2hZC1MjrJG1qGAQSnunFQ.png
[poly-one-to-many]: https://cdn-images-1.medium.com/max/1600/1*uKuKvRNmAQecSGkaFW6Clw.png
[poly-many-to-many]: https://cdn-images-1.medium.com/max/1600/1*B6ZvI_fdPZWWIfJb5rN0kQ.png
