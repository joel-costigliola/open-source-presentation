## Random thoughts as AssertJ open source project lead


---

## Agenda

- A quick presentation of AssertJ 
- Project philosophy
- Community
- Challenges
- Learnings

---

## What is AssertJ?

- java fluent assertions library
- provide assertions for 50+ types: String, List, ... 
- discover assertions with IDE code completion

---

### Who uses it ?

- Orion Health! 
- [The JUnit team](https://twitter.com/marcphilipp/status/1108482731661553664)
- [Spring](https://twitter.com/sam_brannen/status/1131825078256230400)
- Mentioned in [thoughtworks tech radar](https://www.thoughtworks.com/radar/languages-and-frameworks/assertj)

---

### Collection assertions

- Works for array and Stream too

```java
Iterable<Ring> elvesRings = asList(vilya, nenya, narya);
                                                                    
assertThat(elvesRings)                                              
    .hasSize(3)                                                 
    .contains(nenya, narya)                                            
    .doesNotContain(oneRing)                                    
    // need all the elements in the right order
    .containsExactly(vilya, nenya, narya);                      
```

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

---

### Exception assertions

My favorite way: *catchThrowable* 

```java
// GIVEN
ThrowingCallable codeCall = () -> boom("boom!");
// WHEN
final Throwable thrown = catchThrowable(codeCall);
// THEN
assertThat(thrown)
    .isInstanceOf(IllegalStateException.class)
    .hasMessage("boom!");
```

---

### Soft assertions

- Record all assertion errors (don't fail at the first one) 
- Easy soft assertions with JUnitSoftAssertions rule 
- BDD flavor with JUnitBDDSoftAssertions 

```java
@Rule
public JUnitSoftAssertions softly = new JUnitSoftAssertions();

@Test
public void junit_soft_assertions_example() {

  softly.assertThat("Michael Jordan - Bulls")
            .startsWith("Mike")
            .contains("Lakers")
            .endsWith("Chicago");
  // no need to call softly.assertAll() ! :)
}
```

---

Soft assertions errors report:

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

---

### Other features we won't cover

- Conditions
- Assumptions
- AssertJ modules
- Using comparators
- Recursive field by field comparison
- Custom/domain assertions
- modules (DB, Guava)

---

## Why AssertJ?

* I wanted to give back to the community
* It feels good to help people
* I was frustrated with hamcrest
* Forked from Fest Assert

---

## philosophy

* AssertJ is community driven
    * Open to suggestion
    * Ask for community feedback
    * Lower the barrier for contributors
    * Don't pressure contributors
* Documentation is important to help people
    * javadoc, website, assertj-examples

---

## Community

* Leading a community was a social experiment for me
* Always be nice with users
* Be as helpful as possible (github, SO)
* Give credit to contributors
* Acknowledge when you are wrong

---

## Challenges 1/2

* Being responsive
    * acknowledge issues and PR creation
    * often lagging on PR review
* Time consuming 
    * PR review (sometime it is faster to code it than review it)
    * documentation
    * it's ok to take a break from the project

---
## Challenges 2/2

* some people just want their problem solved ASAP
    * be firm and nice (even if you are really annoyed)
* publicize your project (just twitter at the moment)
* assertj.org :(
* handover

---

## Learnings 
 
* You can't please everybody (but you should try)
* Spend your time of valuable issues/PR
    * it's ok to leave minor improvements/bugs behind
* Some PR are difficult to review (too big)
* Specific git flow
* it's ok to take a break from the project