## Random thoughts as an open source project lead


---

## Agenda

- A few word on AssertJ |
- Kickstart |
- Community |
- Challenges |
- Feedback |

---

## What is AssertJ?

- java fluent assertions library
- provide assertions for 50+ types: String, List, ... 
- discover assertions with IDE code completion

```java
List<Ring> elvesRings = asList(vilya, nenya, narya);

assertThat(elvesRings).isNotEmpty()
                      .hasSize(3)
                      .contains(nenya, narya)
                      .doesNotContain(oneRing);
```

---

## Who uses it ?

- Orion Health! 
- The JUnit team https://twitter.com/marcphilipp/status/1108482731661553664
- https://twitter.com/sam_brannen/status/1131825078256230400

Mentioned in thoughtworks tech radar: https://www.thoughtworks.com/radar/languages-and-frameworks/assertj

+++

### PB2 project setup

Adding AssertJ Core to a PB2 Test project:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<ivy-module version="1.0">
  <info module="my.unit.tests" organisation="orchestral" status="integration"></info>
  <configurations>
    <conf name="test"/>
  </configurations>
  <publications>
  </publications>
  <dependencies>
    <dependency conf="test->nodist" name="assertj-core" org="assertj" rev="3.12.0"/>
    <!-- other usual test dependencies -->
    <dependency conf="test->nodist" org="bundle" name="org.junit" rev="4.12.0"/>
    <dependency conf="test->nodist" org="bundle" name="org.mockito" rev="1.9.5"/>
  </dependencies>
</ivy-module>
```

---

## Basic assertions

- Start with Assertions.assertThat(stuffToTest) 
- Discover assertions with code completion 
- Chain assertions 

Note:
Demo: 
* no more confusion about expected vs actual
* assertion description as()
* show representation ?
* IDE config to get assertThat directly
* BDDAssertions for then
* implement WithAssertions 

+++

#### Examples

Basic assertions example:

```java
assertThat("Gandalf")                      
    .as("Check a famous magician name")
    .isNotNull()                       
    .isInstanceOf(String.class)        
    .isNotEqualTo("Saroumane")         
    .isEqualTo("Gandalf");             
```

@[1](`import static org.assertj.core.api.Assertions.assertThat;`)
@[1-2](describe your assertion (to ease understand the error))
@[1-6](chain assertions)

---

## Collection assertions

- Works for Iterable, array and Stream |
- Different "contains" assertion flavors |
- Highlight: extracting and filter features |

Note:
* Stream are converted to List to allow multiple assertions since you only consume a Stream once.
* javadoc example: http://joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractIterableAssert.html#containsExactly-ELEMENT...-
* collections formatting goodies when too many items

+++

#### Examples

```java
Iterable<Ring> elvesRings = asList(vilya, nenya, narya);
                                                                    
assertThat(elvesRings)                                              
    .isNotEmpty()                                               
    .hasSize(3)                                                 
    .contains(nenya, narya)                                            
    .doesNotContain(oneRing)                                    
    // order does not matters nor duplicates                                   
    .containsOnly(nenya, vilya, narya)                          
    // order matters!
    .containsExactly(vilya, nenya, narya);                      
```

@[1-7](common collection assertions)
@[1-9](*containsOnly* expects all the elements and ignores duplicates)
@[1-11](*containsExactly* expects all the elements in the correct order)

+++

#### Extracting feature

```java
// TolkienCharacter fields: name, age, Race
frodo = new TolkienCharacter("Frodo", 33, HOBBIT);
...
boromir = new TolkienCharacter("Boromir", 37, MAN);

fellowshipOfTheRing= list(frodo, sam, merry, pippin, gandalf,
                          legolas, gimli, aragorn, boromir);

assertThat(fellowshipOfTheRing)                       
    .extracting("name") // or use lambda: tc -> tc.getName()
    .contains("Boromir", "Gandalf", "Frodo", "Legolas")
    .doesNotContain("Sauron", "Elrond");               
```
@[1-7](init a list of famous LotR characters)
@[1-12](let's check the names of the fellowshipOfTheRing characters)
@[1-10](create a new List to test with names of fellowshipOfTheRing characters)
@[1-12](assertions on the extracted names)

+++

#### Filter feature

```java
assertThat(fellowshipOfTheRing)                                 
    .filteredOn(tc -> tc.getRace().equals(HOBBIT))               
    .containsOnly(sam, frodo, pippin, merry);

assertThat(fellowshipOfTheRing)                     
    .filteredOn(tc -> tc.getRace().equals(HOBBIT))   
    .extracting(tc -> tc.getName())                  
    .containsOnly("Sam", "Frodo", "Pippin", "Merry");    
```
@[1-3](filterHobbits in fellowshipOfTheRing)
@[5-8](combine filter and extracting FTW !)

Note:
* use of a Java 8 *Predicate* to filter fellowshipOfTheRing

---

## Exception assertions

- Old school way of testing exceptions |
- Better way with assertThatThrownBy |
- Better way with assertThatExceptionOfType  |
- Better way with catchThrowable  |

Note:
* there is a better way / there are better ways!
* JUnit Rule: bad because it puts the assertions before the code
* assertThatCode(() -> {}).doesNotThrowAnyException();

+++

#### Basic exception testing 

```java
void boom() {
  Exception cause = new Exception("red button pushed");
  throw new IllegalStateException("boom!", cause);
}
// Old school way of testing code throwing exceptions
try {                                                              
  boom();                                                        
} catch (final Exception exception) {                           
  assertThat(exception)                                          
      .isInstanceOf(IllegalStateException.class)             
      .hasMessage("boom!");
  return;
}
fail("Should not arrive here");                                          
```
@[1-4](let's test this method)
@[1-14](catch the exception and test it)

Note:
* make the test fail if no exception was thrown

+++

#### Better exception testing (1/3)

Use *assertThatThrownBy* 

```java
assertThatThrownBy(() -> boom())
    .isInstanceOf(IllegalStateException.class)
    .hasMessage("%sm!", "boo")
    .hasCauseInstanceOf(RuntimeException.class)
    .hasStackTraceContaining("red button");
```
@[1](Pass the code to test with a lambda)
@[1-5](Chain exception assertions)

+++

#### Better exception testing (2/3)

Use *assertThatExceptionOfType* 

```java
assertThatExceptionOfType(IllegalStateException.class)
    .isThrownBy(() -> boom())
    .withMessage("boom!")
    .withCauseInstanceOf(RuntimeException.class);

// common exception shortcuts
assertThatIllegalStateException()
assertThatIllegalArgumentException()
assertThatIOException()
assertThatNullPointerException()
```
@[1](Pass the expected exception type)
@[1-2](Use a lambda to pass the code to test)
@[1-4](Chain exception assertions)
@[1-10](Common exceptions shortcuts)

Note:
* common exception shortcuts assertThatIllegalStateException

+++

#### BDD exception testing BDD (3/3)

Use *catchThrowable* 

```java
// GIVEN
ThrowingCallable codeCall = () -> boom();
// WHEN
final Throwable thrown = catchThrowable(codeCall);
// THEN
assertThat(thrown)
    .isInstanceOf(IllegalStateException.class)
    .hasMessage("boom!");
```
@[1-2](GIVEN some code)
@[1-4](WHEN calling the code)
@[1-8](THEN check the caught exception)

Note:
* use `catchThrowableOfType` to get the actual `Throwable` type

---

## Soft assertions

- Record all assertion errors (don't fail at the first one) |
- Easy soft assertions with JUnitSoftAssertions rule |
- BDD flavor with JUnitBDDSoftAssertions |

Note:
Demo: 
* BDD soft assertions

+++

#### Soft assertions example

```java
final SoftAssertions softly = new SoftAssertions();

softly.assertThat("Michael Jordan - Bulls")
          .startsWith("Mike")
          .contains("Lakers")
          .endsWith("Chicago");

// don't forget to call assertAll() !
softly.assertAll();
```

@[1](we need a SoftAssertions wrapper)
@[1-6](record potential assertion failures)
@[1-9](report assertion failures if any)

+++

#### Assertions errors report

```
The following 3 assertions failed:
1) 
Expecting:
 <"Michael Jordan - Bulls">
to start with:
 <"Mike">
2) 
Expecting:
 <"Michael Jordan - Bulls">
to contain:
 <"Lakers"> 
3) 
Expecting:
 <"Michael Jordan - Bulls">
to end with:
 <"Chicago">
```

@[1](number of failures)
@[1-16](numbered list of errors)

+++

#### JUnit Soft assertions example

```java
@Rule
public JUnitSoftAssertions softly = new JUnitSoftAssertions();

@Test
public void junit_soft_assertions_example() {

  softly.assertThat("Michael Jordan - Bulls")
            .startsWith("Mike")
            .contains("Lakers")
            .endsWith("Chicago");

  // no need to call assertAll() ! :)
}
```

---

## Uncovered advanced topics 

- Conditions |
- Assumptions |
- AssertJ modules |
- Using comparators |
- Using recursive field by field comparison |
- Custom/domain assertions |

Note:
- no more confusion about expected vs actual
- assertion description as()
- chaining assertion
- show representation ?

---

## Final words

- Hands on time! 
- Don't hesitate to ask questions

Note:
- What's next for AssertJ: full assertions documentation, better recursive API, java 9 and junit 5 support.

---

## Conditions

- Extend AssertJ assertions with conditions |
- Similar to Hamcrest matchers |
- Combine conditions |
- Using Condition with collections |

Note:
* Stream are converted to List to allow multiple assertions since you only consume a Stream once.

+++

#### Examples

```java
List<String> jedis = asList("Luke", "Yoda", "Obiwan");
List<String> siths = asList("Sidious", "Vader", "Plagueis");
// build condition with a Predicate and a description
Condition<String> jedi = 
        new Condition<>(jedis::contains, "jedi");
Condition<String> sith = 
        new Condition<>(siths::contains, "sith");

assertThat("Yoda").is(jedi);
assertThat("Vader").is(sith).isNot(jedi);

// 'has(condition)' is an alias of 'is(condition)'
Condition<String> jediPowers = new Condition<>(jedis::contains, "jedi powers");
assertThat("Yoda").has(jediPowers);
assertThat("Sponge Bob").doesNotHave(jediPowers);
```

@[1-7](create condition with a Predicate)
@[1-10](assertions using conditions)
@[1-15](*has* is an alias of *is* for readability)

+++

#### Combining conditions

AssertJ provides operators to combine conditions:
- *allOf* : all conditions must be met  
- *anyOf* : at leats one of the conditions must be met 
- *not* : the condition must not be met 
<br>
```java
assertThat("Vader").is(anyOf(jedi, sith))
                     .is(not(jedi));
```

+++

#### Using Condition with collections

```java
assertThat(asList("Luke", "Yoda")).are(jedi);
assertThat(asList("Luke", "Yoda")).have(jediPowers);

assertThat(asList("Chewbacca", "Solo")).areNot(jedi);
assertThat(asList("Chewbacca", "Solo")).doNotHave(jediPowers);

List<String> characters = asList("Luke", "Yoda", "Chewbacca");
assertThat(characters).areAtLeast(2, jedi);
assertThat(characters).haveAtLeast(2, jediPowers);
assertThat(characters).areAtMost(2, jedi);
assertThat(characters).haveAtMost(2, jediPowers);
assertThat(characters).areExactly(2, jedi);
assertThat(characters).haveExactly(2, jediPowers);
```
@[1-2](check that all elements meet the condition)
@[4-5](check that **no** elements meet the condition)
@[7-9](check that at least 2 elements meet the condition)
@[7, 10-11](check that at most 2 elements meet the condition)
@[7, 12-13](check that exactly 2 elements meet the condition)

---

## Assumptions

- Allows to skip tests when assumptions are not met |
- The Assumption API is similar to the Assertion API |

+++

#### Examples

```java
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assumptions.assumeThat;

@Test
public void should_be_skipped_as_assumption_is_not_met() {
  assumeThat(fellowshipOfTheRing).contains(sauron);
  // never executed, the whole test is skipped !
  assertThat(true).isFalse();
}

@Test
public void should_be_executed_as_assumption_is_met() {
  assumeThat(fellowshipOfTheRing).contains(frodo);
  // executed, can succeed or fail like any assertions
  assertThat(frodo.getRace()).isEqualTo(HOBBIT);
}
```

@[2](static import assumeThat from the Assumptions class)
@[3-6,9](an assumption that is never met)
@[3-9](the assertions is not executed as the assumption was not met)
@[11-16](the assertion is executed as the assumption was met)

--- 

## AssertJ modules

AssertJ has modules for:
- Guava: assertj-guava |
- Joda Time: assertj-joda-time |
- Relational DB: assertj-db |
- Neo4J : assertj-neo4j |
- Swing : assertj-swing |

Note:
* only showing guava and DB assertions

+++

#### assertj-guava

Provides assertions for:<br>
 *Multimap*, *Range*, *RangeMap*, *Table* and *Multiset*

```java
import static org.assertj.guava.api.Assertions.assertThat;
import static org.assertj.guava.api.Assertions.entry;

Multimap<String, String> nbaTeams = ArrayListMultimap.create();
nbaTeams.putAll("Lakers", list("Kobe Bryant", "Magic Johnson"));
nbaTeams.putAll("Spurs", list("Tony Parker", "Tim Duncan"));

assertThat(nbaTeams).hasSize(6);
                    .containsKeys("Lakers", "Spurs")
                    .contains(entry("Lakers", "Kobe Bryant"), 
                              entry("Spurs", "Tim Duncan"));
```
Note:
* check the doc at http://joel-costigliola.github.io/assertj/assertj-guava.html

+++

#### assertj-db

Provides a fluent API to test SQL Databases

```java
import static org.assertj.db.api.Assertions.assertThat;

Request request = new Request(source, "select * from albums");

// assertions on a column (partial results for brevety)
assertThat(request).column("title")
    .value().isEqualTo("Boy")
    .value().isEqualTo("October")
    .value().isEqualTo("The Joshua Tree")
    .value().isEqualTo("Zooropa");

// On the values of a row by using the index of the row
assertThat(request).row(1)
    .value().isEqualTo(2)
    .value().isEqualTo(DateValue.of(1981, 10, 12))
    .value().isEqualTo("October");
```
@[1](import assertj-db assertThat)
@[3](let's test this request/query)
@[5-10](check the 'title' column)
@[11-16](check the second row)

Note:
* many other assertions - check the doc at http://joel-costigliola.github.io/assertj/assertj-db.html

---

## Using comparators

Specify your own comparator when evaluating assertions.

```java
// standard comparison : frodo is not equal to sam
assertThat(frodo).isNotEqualTo(sam);

Comparator<TolkienCharacter> raceComparator 
     = (tc1, tc2) -> tc1.getRace().compareTo(tc2.getRace());

assertThat(frodo)
    .usingComparator(raceComparator)
     // succeeds as we compare character's race only
    .isEqualTo(sam)
    .isIn(merry, pippin, sam);
```
@[1-2](TolkienCharacter#equals checks name, age and race)
@[4-5](define a Race comparator)
@[7-8](specify to use the Race comparator)
@[7-11](all subsequent assertions use the Race comparator)

+++

#### Using comparators with collections

Specify an element comparator for collection assertions

```java
// standard comparison : obviously fails !
assertThat(fellowshipOfTheRing).contains(sauron);

// but with race only comparison
assertThat(fellowshipOfTheRing)
    .usingElementComparator(raceComparator)
    // succeeds because Sauron is a Maia like Gandalf
    .contains(gandalf, sauron);
```
@[1-2]()
@[4-6](specify to use a Race comparator on elements)
@[4-8](elements are compared with the Race comparator)

+++

#### Recursive field by field comparison

Useful when: 
- comparing classes not overriding *equals*
- comparing data structures
- supports properties and private fields 

+++

#### Example

```java
Dude jon = new Dude("Jon", 1.8);
Dude sam = new Dude("Sam", 1.5);
jon.friend = sam;
sam.friend = jon;

Dude jonClone = new Dude("Jon", 1.8);
Dude samClone = new Dude("Sam", 1.5);
jonClone.friend = samClone;
samClone.friend = jonClone;

assertThat(jon)
    .isEqualToComparingFieldByFieldRecursively(jonClone);

assertThat(asList(jon, sam))
    .usingRecursiveFieldByFieldElementComparator()
    .contains(jonClone, samClone);
```
@[1-9](data classes without equals)
@[1-12](compare *jon* with *jonClone* field by field recursively)
@[14-16](compare elements field by field recursively)

Note:
- nice error message showing the path and value of non matching fields 
- register comparators by field or type

---

## Custom domain assertions

Express assertions in the domain language: 
- make the assertions code more readable
- domain specific error messages

+++

#### Example

```java
frodo = new TolkienCharacter("Frodo", 33, HOBBIT);

// domain assertion FTW !
assertThat(frodo).hasName("Frodo")
                 .hasAge(33)
                 .hasRace(Race.HOBBIT);
```

Note:
- error message mention the name 

+++

#### Writing domain assertions

- extends *AbstractAssert*
- add a method per assertion

+++?code=src/TolkienCharacterAssert.java&lang=java&title=TolkienCharacter assertions

@[27-28](inherit *AbstractAssert*)
@[51-53](provide an *assertThat* method)
@[63-72](specific assertion for name)

+++

#### Generate domain assertions

- Writing assertions requires (too much) work |
- Generate assertions based on class properties |
- Assertion generator maven plugin |
- Demo |

Note:
- http://joel-costigliola.github.io/assertj/assertj-assertions-generator.html
- works for external libraries
- generics not supported
- best effort to get custom assertions, you might want to enrich them

---

## Final words 

- Give it a try!
- Contact me for help
- Contribute to make it even better

Note:
- What's next for AssertJ: full assertions documentation, better recursive API, java 9 and junit 5 support.

+++

#### poster code

```java
List<TolkienCharacter> hobbits = asList(frodo, sam, pippin, merry);

assertThat(hobbits).hasSize(4)
                   .contains(frodo, sam)
                   .doesNotContain(gandalf, sauron)
                   .allMatch(character -> character.getRace() == HOBBIT)
                   .extracting(TolkienCharacter::getRace)
                   .containsOnly(HOBBIT);
```
