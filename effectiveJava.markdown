#Item 1: Consider static factory methods instead of constructors
 - The static factory method is not the same as the Factory Method pattern in design pattern
 - Advantages
   - they have names
   - they are not required to create a new object each time they're invoked
   - they can return any subtype of their return type
 - Disadvantages
   - the class providing static factory without public or protected constructors cannot be subclassed
   - the static factory method is not readily distinguishable from other static methods

   ```
// Service provider framework sketch

// Service interface
public interface Service {
    ... // Service-specific methods go here
}

// Service provider interface
public interface Provider {
    Service newService();
}

// Noninstantiable class for service registration and access
public class Services {
    private Services() { }  // Prevents instantiation (Item 4)

    // Maps service names to services
    private static final Map<String, Provider> providers =
        new ConcurrentHashMap<String, Provider>();
    public static final String DEFAULT_PROVIDER_NAME = "<def>";

    // Provider registration API
    public static void registerDefaultProvider(Provider p) {
        registerProvider(DEFAULT_PROVIDER_NAME, p);
    }
    public static void registerProvider(String name, Provider p){
        providers.put(name, p);
    }

    // Service access API
    public static Service newInstance() {
        return newInstance(DEFAULT_PROVIDER_NAME);
    }
    public static Service newInstance(String name) {
        Provider p = providers.get(name);
        if (p == null)
            throw new IllegalArgumentException(
                "No provider registered with name: " + name);
        return p.newService();
    }
}
   ```

#Item 2: Consider a builder when faced with many constructor parameters
```
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories      = 0;
        private int fat           = 0;
        private int carbohydrate  = 0;
        private int sodium        = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
            { calories = val;      return this; }
        public Builder fat(int val)
            { fat = val;           return this; }
        public Builder carbohydrate(int val)
            { carbohydrate = val;  return this; }
        public Builder sodium(int val)
            { sodium = val;        return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```
Note NutritionFacts is immutable, and that all parameter default values are in a single location. The builder's setter methods return the builder itself so that invocations can be chained. Here's how the client code looks:
```
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).
  calories(100).sodium(35).carbohydrate(27).build();
```
# Item 3: Enforce the singleton property with a private constructor or an enum type
```

// Method(1) Singleton with public final field,  the declarations make it clear that the class is a singleton
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}

//Method(2), static factory, we can change the instance generated without changing api
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}

//Method (3) readResolve method to preserve singleton property
private Object readResolve() {
     // Return the one true Elvis and let the garbage collector
     // take care of the Elvis impersonator.
    return INSTANCE;
}

//Method (4)  Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```
#Item 4: Enforce noninstantiability with a private constructor
to write a class that is just a grouping of static methods and static fields in the manner of java.lang.Math or java.util.Arrays.Such utility classes were not designed to be instantiated
```
class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
    ...  // Remainder omitted
}
```
#Item 5: Avoid creating unnecessary objects
An object can always be reused if it is immutable
```
public class Person {
    private final Date birthDate;
    // Other fields, methods, and constructor omitted

    /**
     * The starting and ending dates of the baby boom.
     */
    private static final Date BOOM_START;
    private static final Date BOOM_END;

    static {
        Calendar gmtCal =
            Calendar.getInstance(TimeZone.getTimeZone("GMT"));
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_START = gmtCal.getTime();
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_END = gmtCal.getTime();
    }

    public boolean isBabyBoomer() {
        return birthDate.compareTo(BOOM_START) >= 0 &&
               birthDate.compareTo(BOOM_END)   <  0;
    }
}
```
#Item 6: Eliminate obsolete object references
Whenever an element is freed, any object references contained in the element should be nulled out.
Another common source of memory leaks is caches
Third common source of memory leaks is listeners and other callbacks. 
#Item 7: Avoid finalizers
#Item 8: Obey the general contract when overriding equals
 recipe for a high-quality equals method
1. Use the == operator to check if the argument is a reference to this object.
2. Use the instanceof operator to check if the argument has the correct type. 
3. Cast the argument to the correct type.
4. For each "significant" field in the class, check if that field of the argument matches the corresponding field of this object.
5. When you are finished writing your equals method, ask yourself three questions: Is it symmetric? Is it transitive? Is it consistent?
    Reflexive: For any non-null reference value x, x.equals(x) must return true.
    Symmetric: For any non-null reference values x and y, x.equals(y) must return true if and only if y.equals(x) returns true.
    Transitive: For any non-null reference values x, y, z, if x.equals(y) returns true and y.equals(z) returns true, then x.equals(z) must return true.
    Consistent: For any non-null reference values x and y, multiple invocations of x.equals(y) consistently return true or consistently return false, provided no information used in equals comparisons on the objects is modified.
    For any non-null reference value x, x.equals(null) must return false.
#Item 9: Always override hashCode when you override equals
#Item 10: Not override clone
#Item 11: Always override toString
#Item 12: Consider override Comparable
If you are writing a value class with an obvious natural ordering, such as alphabetical order, numerical order, or chronological order, you should strongly consider implementing the interface:
public interface Comparable<T> {
    int compareTo(T t);
}
#Item 13: Minimize the accessiblity of class and members
#Item 14: In public classes, use accessor methods, not public fields
#Item 15: Minimize mutability
    1. Don't provide any methods that modify the object's state 
    2. Ensure that the class can't be extended. 
    3. Make all fields final.
    4. Make all fields private. 
    5. Ensure exclusive access to any mutable components.
#Item 16: Favor composition over inheritance
#Item 17: Design and document for inheritance or else prohibit it
#Item 18: Prefer interfaces to abstract classes 
