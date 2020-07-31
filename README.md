# Pumpkin Test Framework
*Another test framework for C++ apps*

Pumpkin Test Framework is designed to provide an efficient way to write unit & integration tests for C++ apps.
It's a (very) small header file only library written in C++ 11 and the standard library. So, it's easy to use and to adapt to your own way of working.

## How to use it


* A test category is represented by a class inheriting from `PumpkinTest::AutoRegisteredTestFeature`.
* A Single test is represented by a name and a lambda passed to the method `PumpkinTest::AutoRegisteredTestFeature::test(std::string, std::function<void()>)`
* A test is considered as OK if no `PumpkinTest::PumpkinTestException` is raised during the execution
* Pumpkin Test Framework provides several assertions to check the results

Let's say we want to test the following function:

```
std::string fizzbuzz(int value)
{}
```

First, we have to write our unit tests as followed in a source file (such as `tst_fizzbuzztest.cpp`):

```
class FizzbuzzTest: public PumpkinTest::AutoRegisteredTestFeature
{
public:
    FizzbuzzTest(): PumpkinTest::AutoRegisteredTestFeature("fizzbuzz() function Unit Tests")
	{
	    test("Should return 'fizz' if parameter is a multiple of 3", []()
		{
		    PumpkinTest::Assertions::assertEquals(std::string("fizz"), fizzbuzz(3));
		});

        test("Should return 'buzz' if parameter is a multiple of 5", []()
		{
		    PumpkinTest::Assertions::assertEquals(std::string("buzz"), fizzbuzz(5));
		});

        test("Should return 'fizzbuzz' if parameter is a multiple of 3 and 5", []()
		{
		    PumpkinTest::Assertions::assertEquals(std::string("fizzbuzz"), fizzbuzz(15));
		});

        test("Should return the raw value if parameter is not a multiple of 3 or 5", []()
		{
		    PumpkinTest::Assertions::assertEquals(std::string("7"), fizzbuzz(7));
		});
	}
};
```

Then, we have to register our `FizzbuzzTest` as a test class by using `REGISTER_PUMPKIN_TEST`


```
class FizzbuzzTest: public PumpkinTest::AutoRegisteredTestFeature
{
...
};

REGISTER_PUMPKIN_TEST(FizzbuzzTest)
```

The `REGISTER_PUMPKIN_TEST` will just generate an instance of `FizzbuzzTest` in order to register it in the list of test features runned by Pumpkin Test Framework.


Last step: run the tests in your main():

```
int main(int argc, char *argv[])
{
    return PumpkinTest::runAll();
}
```

## Report

Pumpkin Test Framework will generate a text report once all the tests are played:

```
fizzbuzz() function Unit Tests | Should be equal                                                      | OK
                               | Should return 'buzz' if parameter is a multiple of 5                 | OK
							   | Should return 'fizzbuzz' if parameter is a multiple of 3 and 5       | OK
							   | Should return the raw value if parameter is not a multiple of 3 or 5 | OK

Summary: 4 tests passed
```

If a test is KO, the report will display the cause:

```
fizzbuzz() function Unit Tests | Should be equal                                                      | OK
                               | Should return 'buzz' if parameter is a multiple of 5                 | OK
							   | Should return 'fizzbuzz' if parameter is a multiple of 3 and 5       | OK
							   | Should return the raw value if parameter is not a multiple of 3 or 5 | KO Cause: Expected value was '7' but actual value is '4'

Summary: 3 tests passed, 1 test failed
```

If all tests are OK, `Pumpkin::runAll()` will return 0. If at least one test is KO or Failed, it will return -1

## Adapt Pumpkin Test framework

### Assertions and custom types
The basic assertions can be used with custom type as long as they define:
* operator<< for std::ostream (used to print a message in case of error)
* Equality operator for `PumpkinTest::Assertions::assertEquals`

For example:

value.h
```
class Value
{
public:
    Value(int u): value(u)
	{}
	bool operator==(Value const& other)
	{
	    return other.value == this->value;
	}
	int const value;
};
```

tst_value.cpp
```
inline std::ostream& operator<<(std::ostream& os, Value const& s)
{
    os << "Value(" << s.value <<")";
	return os;
}

class ValueTest: public PumpkinTest::AutoRegisteredTestFeature
{
public:
    FizzbuzzTest(): PumpkinTest::AutoRegisteredTestFeature("Value class Unit Tests")
	{
	    test("Should be equal", []()
		{
		    PumpkinTest::Assertions::assertEquals(Value(2), Value(2));
		});
};

REGISTER_PUMPKIN_TEST(ValueTest)
```

main.cpp
```
int main(int argc, char *argv[])
{
    return PumpkinTest::runAll();
}
```

### Custom assertions

Pumpkin Test Framework can handle custom assertions:
* If a `PumpkinTest::PumpkinTestException` (or any derived class) is raised during the execution, the test is KO
* If another type of exception is raised during the execution, the test is FAILED
* Otherwise, the test is OK

You can create your own assertion based these rules:

```
class NotGreaterException: public PumpkinTestException {
public:
    NotGreaterException(int left, int right):
	    PumpkinTestException(
		(
		    std::stringstream() << left << "should be greater than " << right).str()
		)
	{}
};

int void assertGreater(int left, int right)
{
    if (!(left > right))
	    throw PumpkinTest::exceptions::NotGreaterException(left, right);
}


test("5 should be greater than 3", []()
{
    PumpkinTest::Assertions::assertGreater(5, 3);
});

The message passed to PumpkinTestException constructor will be displayed as the cause in the report.

```
