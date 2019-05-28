## Random thoughts as AssertJ open source project lead


---

## Agenda

- A quick presentation of AssertJ 
- Community 
- Challenges 
- Feedback 

---

## What is AssertJ?

- java fluent assertions library
- provide assertions for 50+ types: String, List, ... 
- discover assertions with IDE code completion

---

### Who uses it ?

- Orion Health! 
- The JUnit team https://twitter.com/marcphilipp/status/1108482731661553664
- https://twitter.com/sam_brannen/status/1131825078256230400
- Mentioned in thoughtworks tech radar: https://www.thoughtworks.com/radar/languages-and-frameworks/assertj

---

### Collection assertions

- Works for array and Stream too

```java
Iterable<Ring> elvesRings = asList(vilya, nenya, narya);
                                                                    
assertThat(elvesRings)                                              
    .hasSize(3)                                                 
    .contains(nenya, narya)                                            
    .doesNotContain(oneRing)                                    
    // need all the lement in the right order
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
ThrowingCallable codeCall = () -> boom();
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


#### Soft assertions example

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

---

### Other features we won't cover

- Conditions 
- Assumptions
- AssertJ modules
- Using comparators
- Using recursive field by field comparison
- Custom/domain assertions

## Community 

## Challenges 

## Learnings 
