# Triple Pivot

A way to link 3 many-to-many relations together in แก้สำหรับ Laravel 5 แล้ว's Eloquent.

---

## Contents

### Usage

### Setup

1. Require in `composer.json` and `config/app.php`
2. Add the trait in all 3 models
3. Define the relation method as `->tripleBelongsToMany()`
4. (Optional) ในกรณีที่ต้องการเปลี่ยนไม่เรียกฟังก์ชัน `->third()`

---

## Usage

	// Get the first Employee, and autoload their positions
	$emp = Employee::with( 'positions' )->first();

	// Or find an employee and loads all their positions or division
	$emp = Employee::find(1);
	$emp->positions;
	$emp->divisions;

	// Get attributes
	$emp->positions->first()->name; // Get first Postion associate with 
	$emp->positions->first()->third->first()->name;

	// Like an ordinary belongsToMany
	$emp->positions->first(); // Position model

	// Get the division associated with a given position for the employee
	$emp->positions->first()->third; // Division model
	$emp->positions->first()->division; // Division model (only if you did step 4)

	// Attach a position/division to a employee
	$emp->positions()->attach( [ $posId, $divId ] ); // Pass an array of 2 IDs
	$emp->positions()->attach( [ Position::find( $posId ), Division::find( $divId ) ] ); // Pass an array of 2 models

	//สำหรับ Position ตอนนี้ใช้ employees กับ divisions
	//สำหรับ Division ตอนนี้ใช้ employees กับ positions

---

## Setup

### 1.Require the package

**composer.json**: Add in the package definition.

	"require": {
		...
        "illuminate/support": "5.1.*"
    },
	...
    "autoload": {
    	...
		"psr-0": {
			"Jarektkaczyk\\TriplePivot": "src/"
		}
    }

Run `composer update` in the Terminal.

### 2. Add the trait into all of our models

**Models/Employee.php**: Two `use` statements - one to pull in the namespaced Trait, one to use it in the Model.

	<?php
	
	namespace Choice;
	use Illuminate\Database\Eloquent\Model;
	use Jarektkaczyk\TriplePivot\TriplePivotTrait;
	
    class Employee extends Model {
    	use TriplePivotTrait;
    }

**Models/Position.php**: As above

	<?php

	namespace Choice;
	use Illuminate\Database\Eloquent\Model;
	use Jarektkaczyk\TriplePivot\TriplePivotTrait;
	
	class Position extends Model {
    	use TriplePivotTrait;
	}

**Models/Division.php**: As above

	<?php
	
	namespace Choice;
	use Illuminate\Database\Eloquent\Model;
	use Jarektkaczyk\TriplePivot\TriplePivotTrait;
	
	class Division extends Model {
    	use TriplePivotTrait;
	}

### 3. Define the `tripleBelongsToMany` relation

**Models/Employee.php**

	<?php
	
	namespace Choice;
	use Illuminate\Database\Eloquent\Model;
	use Jarektkaczyk\TriplePivot\TriplePivotTrait;
	
    class Employee extends Model {
    	use TriplePivotTrait;
    	
		/**
		 * @return \Jarektkaczyk\TriplePivot\TripleBelongsToMany
		 */
	    public function positions()
	    {
	    	// for order of keys --------------- first Model---------second Model------------------------------------- own_id ---- firstModel_id--secondModel_id  
	    	return $this->tripleBelongsToMany( 'Choice\Position', 'Choice\Division', 'division_employee_position', 'employee_id', 'position_id', 'division_id' );
	    }
	    public function divisions()
	    {
	    	return $this->tripleBelongsToMany( 'Choice\Division', 'Choice\Position', 'division_employee_position', 'employee_id', 'division_id', 'position_id' );
	    }
    }

### 4. (Optional) Define a nicer method than `->third` on `Models/Position`

**Models/Position.php**: Create a new method `getDivisionAttribute()` which forwards on to the `getThirdAttribute()` method so we can call `$position->division->name` instead of `$position->third->name`.

	<?php

	namespace Choice;
	use Illuminate\Database\Eloquent\Model;
	use Jarektkaczyk\TriplePivot\TriplePivotTrait;
	
	class Position extends Model {
    	use TriplePivotTrait;

		/**
		 * @return \Illuminate\Database\Eloquent\Model
		 */
		public function getDivisionAttribute() {
			return $this->getThirdAttribute();
		}
	}
