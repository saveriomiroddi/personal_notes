# Beginning C++ Through Game Programming

- [Beginning C++ Through Game Programming](#beginning-c-through-game-programming)
  - [Chapter 1](#chapter-1)
  - [Chapter 2](#chapter-2)
  - [Chapter 3](#chapter-3)
  - [Chapter 4](#chapter-4)
  - [Chapter 5](#chapter-5)
  - [Chapter 6](#chapter-6)
  - [Chapter 7](#chapter-7)
  - [Chapter 8](#chapter-8)
  - [Chapter 9](#chapter-9)
  - [Chapter 10](#chapter-10)

(See `EXTRA` for things not in the book)

## Chapter 1

```cpp
// names in brackets are libraries included with the compiler
#include <iostream>

// stuff from iostream
std::cout << std::endl
std::cin >> myvar

// include namespace:
using namespace std;

// or, use "declarations":
using std::cout;
using std::endl;

// result:
cout << endl;

// some data types
int score = 0; // defaults to signed
unsigned long veryHighScore; // signedness applies only to integers
short lives, aliensKilled;
float average;
double distance;
char playAgain = 'a'; // 1 byte
bool shieldsUp; // `true` or `false`

// typedef!
typedef unsigned short int ushort;

// some operators
score += 9;
score++ * 10; // 90
++score * 10; // higher priority: 110

//integer wrap around!!
unsigned int score = 4294967295;
cout << ++score; // 0

// constant
const int ALIEN_POINTS = 150;

// enum
// when undefined, get the previous value + 1; they start from 0.
enum difficulty {NOVICE, EASY, NORMAL, HARD, UNBEATABLE};
difficulty currentDifficulty = EASY;
cout << currentDifficulty; // 1

// will give the same value to BOMBER and CRUISER!
enum shipCost {FIGHTER = 25, BOMBER, CRUISER = 26};
shipCost currentcost = EASY;
cout << currentcost; // 26

// string
// `cin` supports strings, but only without whitespace; it also automatically
// casts to the variable type!
std::string myString;
cin >> myString;

// off book: use getline() to read full lines (including whitespace); don't
// forget to call ignore() first. 
cin.ignore();
getline(cin, newGameTitle);

// string to number conversion (EXTRA)
// see https://en.cppreference.com/w/cpp/string/basic_string/stof.
stof(myString[, position])
```

## Chapter 2

```cpp
// Ints > 0 evaluate to true
if (score) { cout << "Scored some points!" << endl }

// switch/case. **don't forget the break!!**
// only applies to int (and enum/char)
switch (score)
{
  case 0:
    cout << "No points!" << endl;
    break;
  case 1:
    cout << "One point!" << endl;
    break;
  case 2:
  {
    // use a block in order to define a variable here (EXTRA)
    int satanPoints = 666;
    count << "Satan's points: " << satanPoints << endl;
    break;
  }
  default:
    count << "Some points!" << endl;
}

// while...do
while (condition) {}

// do...while
do {} while (condition);

// loop modifiers
break;    // exit a loop
continue; // skips to the next iteration

// conditionals
true || false && false // true: `&&` has highter priority than `||`

// random, date, casting
// date: see https://en.cppreference.com/w/cpp/chrono/c/time
#include <cstdlib> // random APIs
#include <ctime>   // time

srand(static_cast<unsigned int>(time(nullptr))); // seed rng
int randomNumber = rand();
```

## Chapter 3

```cpp
<instance>.<method> // `.`: member selection operator

// strings instantiation
string myString = "01234567";
string myString2("Over");
string myString3(3, 'z'); // "zzz"; EXTRA: conversion from char to string

// string APIs
// they are dynamically sizeable. !! THERE ARE NO BOUNDARY CHECKS !!
myString.size()
myString.length()
myString += myString2
myString[0]          // nth byte (not character!)
myString[0] = "x"    // only one character allowed
myString.erase([<start>[, <end>]]) // positions are included; <end> defaults to last pos.
                                   // no args erases the entire string
myString.empty()     // test
myString + myString2 // concatenation

// substring search.
// watch out: npos is not -1; it's actually the maximum string size
if (myString.find("tri") == string::npos) cout << "not found" << endl;
myString.find("012", 1) // start position

// arrays
const int MAX_ITEMS = 10 // good practice ->
string inventory[MAX_ITEMS]  // -> for array instantiation
string inventory[] = {"sword", "gun" } // size = 2
string inventory[MAX_ITEMS] = {"sword", "gun" } // define the first 2; the others are undefined
int matrix[10][10]                   // bidimensional array
int matrix2[][] = { {0, 1}, {2, 3} } // inline; requires the sizes

// C-style strings
char myString4[] = "abc"  // + null terminator; has no member functions!
char myString5[5] = "abc" // 5 + null terminator
// They can be generally mixed with strings; the result of a mixed concatenation is always a
// string.
string result = myString + myString4

// Other APIs (chapter 4)
int charPos = "string".find("r"); // string::npos if not found
char upChar = toupper('a')
```

## Chapter 4

```cpp
// Vectors
#include <vector>
std::vector<string> inventory;
inventory.push_back("sword");
inventory[0] = "longer sword"; // WATCH OUT! no boundary check
for (unsigned int i = 0; i < inventory.size(); ++i) cout << inventory[i] << endl;
inventory.pop_back();
inventory.clear();
if (inventory.empty()) cout << "Inventory empty!" << endl;

// Vector initializers
std::vector<string> inventory(10);           // 10 elements
std::vector<string> inventory(3, "hoorray"); // 3*"hoorray"
std::vector<string> inventory(otherVector);  // copy from another vector
```

Iterators:

```cpp
// Iterators: standard and constant (allows no change)
std::vector<string>::iterator s_iter;
std::vector<string>::const_iterator c_iter;

// Replace an entry
s_iter = inventory.begin();
*s_iter = "a different axe";

// Whooa!! Use `vector.begin()` for referencing the first element
inventory.insert(iter, "crossbow");
inventory.erase(iter + 2);

// Iterate. Note that end() is *not* the last element.
// Not safe if the vector is modified!!
for (iter = inventory.begin(); iter != inventory.end(); ++iter)
{
  unsigned int wordSize = (*iter).size(); // or iter->size()
  cout << *iter << " (" << wordSize << ")" << endl;
}
```

Algorithms:

```cpp
// STL algorithms work with anything supporting the required interface, even
// strings (use `begin()` and `end()`).
#include <algorithm>

// Doesn't assume an ordered vector
std::vector<int>::const_iterator iter = std::find(scores.begin(), scores.end(), 66);
if (iter != scores.end()) cout << "found!" << endl;

// Randomizer a vector
srand(static_cast<unsigned int>(time(0)));
std::random_shuffle(scores.begin(), scores.end());

// Sort a vector
std::sort(scores.begin(), scores.end());

// Returns an iterator pointing to the first element in the range [first, last) that is greater than value.
// Use for keeping a vector sorted.
std::upper_bound(games.begin(), games.end(), newGameTitle);
```

Memory allocation:

```cpp
myVector.capacity(); // current capacity (*not* size!)
myVector.reserve(20); // increase capacity to 20
```

Containers:

vector         | sequential  |
deque          | sequential  | double ended queue
list           | sequential  | linear list
map            | associative |
multimap       | associative | multiple values per key possible
set            | associative |
multiset       | associative | each element is not necessarily unique
queue          | adaptor     | FIFO
priority_queue | adaptor     | supports finding and removing high priority element
stack          | adaptor     | LIFO

## Chapter 5

Prototypes: should be generally used. They're mandatory when a function is defined after being called:

```cpp
void myFunction();

int main() { myFunction(); }

void myFunction() { }
```

Blocks (see previous use in switch/case):

```cpp
int main()
{
  myCall();
  {
    myCallInANewScope();
  }
}
```

Global variables/constants:

```cpp

int globalVar = 10;
const int GLOBAL_CONST = 666;

int main() { ... }; 
```

Default arguments:

```cpp
void myFunction(int param1, int param2 = 2)
```

Overloading functions:

```cpp
int returnValue(int param);
string returnValue(string param);
```

Inline functions (done in the definition, not the declaration):

```cpp
inline int inlinedFunction() {}
```

## Chapter 6

Create a reference to a variable (*different* from pointer (`*`)):

```cpp
  int myVar = 1000;
  int& myVarRef = myVar;

  cout << myVarRef << endl; // prints the myVar value
  myVarRef += 500;          // !! increases myVar !!
  myVarRef = anotherVar     // !! UNEXPECTED: sets myVar value to anotherVar, and retains the reference !!
```

references *must* be assigned to a variable on initialization (the variable doesn't necessarily need a value).

A regular variable can be passed to a function that takes a reference:

```cpp
void swap(int& x, int& y) {
  int temp = x;
  x = y;
  y = temp;
}

int main() {
  int x = 0, y = 10;
  swap(x, y);
}
```

Passing by reference is faster than passing by value, since the value is not copied.

Const references can't be changed; !! even some methods are not permitted !!!

```cpp
void displayVector(const vector<string>& vec) {
  vec.push_back("new value"); // !! FAILS !!
}
```

References can be returned:

```cpp
string& extractEntry(vector<string> vec) {
  return vec[1];
}
```

WATCH OUT!!! Never return references to local varibles, as they're temporary:

```cpp
string& returnString() {
  string newString = "abc";
  return newString;
}
```

## Chapter 7

```cpp
int* pScore = nullptr;  // good practice to assign it to nullptr
int score = 1000;
pScore = &score;

cout << "&score: " << &score << endl;   // `score` address
cout << "pScore: " << pScore << endl;   // `pScore` value (= `score` address)
cout << "&pScore: " << &pScore << endl; // `pScore` address
cout << "*pScore: " << *pScore << endl; // score value

*pScore += 500; // operates on the pointed value (`score`) (`*` = dereference operator)

string text = "pizza";
string* pText = &text;

cout << pText->size() << endl; // `->` is operator for `(*<ptrVar>).`

int* const score; // constant pointer; can change the pointed value
const int* score; // !! WATCH OUT !! pointer to constant - can't change the pointed value
const int* const score // constant pointer to constant (!!)
```

An array is actually a constant pointer to the first element of an array.

One of the consequences is that dereferencing an array returns the first element:

```cpp
int nums[3] = { 0, 2, 4};
cout << *nums << endl;
```

As with pointers in general, arithmetic can be used:

```cpp
cout << *(nums + 2) << endl; // prints element 2
```

## Chapter 8

Class definition(s)!:

```cpp
class Critter
{
public:
  static int GetTotal();      // static function

  // when we define a constructor, the compiler won't provide a default constructor anymore.
  // note that we set the default only in the constructor prototype
  Critter(int hunger = 10);
  void Greet();
  int getHunger() const;      // `const`: can't modify data member, or call a non-const function

private:
  static int s_Total;         // static var; not initialization allowed here, unless const (!)

  int m_Hunger;
};                            // needs closing semicolon

int Critter::s_Total = 0;     // !! static var initialization

int Critter::GetTotal()       // no need to specify "static"
{
  return s_Total;
}

Critter::Critter(int hunger):
  m_Hunger(hunger)            // !! assigns `hunger` to `m_Hunger`
{
  ++s_Total;
}

void Critter::Greet()
{
  cout << "Hello, I'm a critter! My hunger level is: " << m_Hunger << endl;
}

int Critter::getHunger() const
{
  return m_Hunger;
}

int main()
{
  Critter critter1(5), critter2;

  critter1.Greet();

  cout << "Total population: " << Critter::GetTotal() << endl;

  // ...
}
```

## Chapter 9

Friend functions && operator overloading!

```cpp
class Critter
{
  // The parameter for the friend class *must* be const!!
  friend void PrintName(const Critter& critter);
  friend ostream& operator<<(ostream& os, const Critter& critter);

  // ...

private:
  string m_Name;
};

void PrintName(const Critter& critter)
{
  cout << critter.m_Name << endl;
}

ostream& operator<<(ostream& os, const Critter& critter)
{
  os << critter.m_Name;
  return os;
}

int main()
{
  cout << Critter("Billy") << endl;
  // ...
}
```

Dynamic memory allocation!!

```cpp
void leakingFunction()
{
  int* drip1 = new int(30);
}

int main()
{
  // dynamic allocation, without assignment, and deallocation
  int* pHeap = new int;
  *pHeap = 10;
  delete pHeap;

  // dynamic allocation, with assignment
  int* pHeap2 = new int(20);
  delete pHeap2;

  leakingFunction();

  // LEAK! overwritten, without freeing
  int* drip2 = new int(50);
  drip2 = new int(100);
  delete drip2;

  // Unclear example; it seems to show how deallocated pointers should be nullified for
  // safety, although it doesn't make sense to do that before a `return`.
  pHeap = nullptr;
  pHeap2 = nullptr;

  return 0;
}
```

Interesting use cases of memory de/allocation, and `this`:

```cpp
class Critter
{
public:
  Critter(const string& name);
  ~Critter();                           // destructor!!
  Critter(const Critter& c);            // copy constructor
  Critter& operator=(const Critter& c); // overloaded Critter= op

private:
  string* m_pName;
};

Critter::Critter(const string& name)
{
  m_pName = new string(name);
}

Critter::~Critter()
{
  delete m_pName;
}

Critter::Critter(const Critter& c)
{
  m_pName = new string(*(c.m_pName));
}

Critter& Critter::operator=(const Critter& c)
{
  // `this` is a reference (`Critter* const`)
  if (this != &c)
  {
    delete m_pName;
    m_pName = new string(*(c.m_pName));
  }

  return *this;
}

int main()
{
  Critter critter1("pippo");
  Critter critter2(critter1);

  critter1 = critter2;

  // destructors are called here
  return 0;
}
```
// EXTRA: lvalue = something that has an address; rvalue = something that hasn't
// think of assignment <l>eft/<r>right

## Chapter 10

Subclassing:

```cpp
class Boss : public Enemy
{}
```

WATCH OUT: Although constructors and destructors are not inherited from a base class, they are called when an instance is created/destroyed.

Protected access level:

```cpp
class Enemy
{
// [...]

protected:
  int m_Damage;
};
```

protected members are not accessible outside of the class, except in some cases of inheritance

Using public derivation:

```cpp
class Boss : public Enemy
{
public:
  Boss::Boss();
};

Boss::Boss():
  Enemy(damage)            // call base class constructor
{
  // ...
}
```

implies that:

- public members in the base class    -> public members in the derived class
- protected members in the base class -> protected members in the derived class
- private members in the base class   -> inaccessible in the derived class

protected/private derivation are rare, and outside the book scope.

Virtual methods, and overriding:

```cpp
class Enemy
{
public:
  // ...
  void virtual Attack() const;
};

void Enemy::Attack() const
{
  cout << "Perform base attack";
}

class Boss : public Enemy
{
public:
  // ...
  void virtual Attack() const;     // `virtual` is optional here
};

void Boss::Attack() const
{
  Enemy::Attack();                 // call base class member function
  cout << " and an extra attack." << "\n";
}
```

WATCH OUT!: It's good practice to declare `virtual` the functions intended to be overridden; not doing so may have unintended side effects (see Chapter 10: Polymorphism)

Overloading an operator in a derived class:

```cpp
Boss& Boss::operator=(const Boss &b)
{
  Enemy::operator=(b); // call the operator in the superclass

  return *this;
}
```

Copy constructur of a derived class (nothing new):

```cpp
class Boss : public Enemy
{
public:
  // ...
  Boss(Boss& b);
;}

Boss::Boss(Boss& b):
Enemy(b)
{
  // ...
}
```

Polymorphism:

```cpp
Enemy* pBadGuy = new Boss();
pBadGuy->Attack();
```

WATCH OUT! If you override a non-virtual member function in a derived class and call that member function on a derived class object through a pointer to a base class, you’ll get the results of the base class member function and not the derived class member function definition.

WATCH OUT! The above applies to destructors as well, so destructors should be made virtual for classes with subclasses.

Abstract classes:

```cpp
class Creature
{
public:
  virtual void Greet() = 0;   // Pure virtual member function: makes the class abstract
  // ...
};
```
