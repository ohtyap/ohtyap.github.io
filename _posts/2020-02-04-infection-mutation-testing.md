---
title: Mutation testing with Infection
categories: [php, testing]
description: How reliable are your tests? Are they covering all branches? How confident are you about them when code changes?
---

The most important aspect about writing tests, is the confidence that everything is still working after
your source code has been modified. Tests ensure everything still works as originally intended without breaking 
any existing functionality while creating updates, adding new features or refactoring the code base. 

One metric we can use to measure tests is *Code Coverage* - or more specific *Line Coverage*. Line coverage gives 
the developer a rough overview which part of the code is already tested and which part is still missing tests - 
especially in combination with the `@covers` annotation which ensure that code is only marked as covered when 
a dedicated test exist. But line coverage has some limitations and is not a good metric to measure the quality of the 
tests or if all branches of the code are really covered. Lets have a look at the following example:

```php
class SomeClass
{
    public function check(int $a, int $b): bool
    {
        if (($b > 50) && ($a < 10 || $a > 90)) {
            return false;
        }
        
        return true;
    }
}
```

```php
class SomeClassTest extends TestCase
{
    public function testCheck()
    {
        $someClass = new SomeClass();
        $this->assertFalse($someClass->check(5, 60));
        $this->assertTrue($someClass->check(95, 40));
    }
}
```
This test has 100% coverage. But `$a > 90` was never executed - line coverage does not provide this information. 
Here comes mutation testing into play. 

### Basics of Mutation Testing

Because the above test does not cover `$a > 90` you could change this part of code to something else (or even remove it) 
and your tests would still pass. Wrong/incomplete code should be found by your test suite - thats why you wrote your 
tests in the first place. Thats exactly how mutation testing works: It modifies your source code in small ways - 
called Mutation - and runs the changed code - called Mutant - against your test suite. If your tests fail, great 
your tests are covering this situation - you killed the mutant. If your tests still pass (like in the example above) 
it indicates that your tests doesn't cover this particular situation.

### Infection

[Infection](https://infection.github.io/){:target="_blank"} is a mutation testing framework for PHP. Its very easy 
to get started as you just need to install a package via composer:

```shell
composer require --dev infection/infection
```

Afterwards just run infection:
```shell
vendor/bin/infection
```

The first time Infection runs, it will ask you several questions and creates a basic `infection.json.dist` config file. More
configuration options can be found [here](https://infection.github.io/guide/usage.html#Configuration){:target="_blank"}.
The output will look similar to this:

```shell
11 mutations were generated:
       7 mutants were killed
       0 mutants were not covered by tests
       4 covered mutants were not detected
       0 errors were encountered
       0 time outs were encountered

Metrics:
         Mutation Score Indicator (MSI): 63%
         Mutation Code Coverage: 100%
         Covered Code MSI: 63%
```

Explained in detail:
- X mutants were killed: How much code changes/mutants - possible errors - are detected by your tests. (positive outcome)
- X mutants were not covered by tests: Infection changes a part of code which is not covered by any test. This might happen
when you do not have 100% line coverage. (negative outcome)
- X covered mutants were not detected: This is the counterpart of the killed mutants - your tests are incomplete 
(negative outcome)
- X errors were encountered: Some changes to the source code by infection might lead to a fatal error. This is a positive 
result and such mutant is not considered as escaped. (positive outcome)
- X time outs were encountered: Similar to errors, mutations might lead to endless loops and time out. This is a positive 
result and such mutant is not considered as escaped. (positive outcome)

Based on this numbers several metrics can be calculated:


*Mutation Score Indicator (MSI)*
```shell 
TotalKilled = KilledCount + TimedOutCount + ErrorCount;
MSI = (TotalKilled / TotalMutantsCount) * 100;
```

*Mutation Code Coverage*
```shell 
TotalCovered = TotalMutantsCount - NotCoveredByTestsCount;
CoveredRate = (TotalCovered / TotalMutantsCount) * 100;
```

*Covered Code MSI*
```shell 
TotalCovered = TotalMutantsCount - NotCoveredByTestsCount;
TotalKilled = KilledCount + TimedOutCount + ErrorCount;
CoveredCodeMSI = (TotalKilled / TotalCovered) * 100;
```

After runnining infection a file called `infection.log` is created (you might want to change the file location to 
something else in your configuration file). In this file we can see the actual mutations and which mutation operator was
used. A full list of all mutation operators provided by infection can be found 
[here](https://infection.github.io/guide/mutators.html){:target="_blank"}

### Conclusion

Infection (and mutation testing in general) is a very useful tool for improving the quality of the project’s test suite.
Its not hard starting using with Infection. You just need one more configuration file and install the dependency via 
composer and you are ready to go. Just give it a try!