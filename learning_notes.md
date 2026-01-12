# Why Use Interfaces in Java?
- Interfaces define contracts - they specify what methods a class must implement, but not how.
- Using interfaces lets classes like A depend on abstractions (interfaces) instead of concrete classes (B).
- This means you can swap implementations easily without changing the dependent class A. For example, replacing B with C that implements the same interface.
- Without interfaces, changing implementations requires updating A's code (like changing the type of its dependency), which can be cumbersome and error-prone.
- Using interfaces also:
  - Helps with unit testing (by allowing mock implementations).
  - Supports loose coupling, making code easier to maintain and extend.
  - Aligns with SOLID principles, especially Dependency Inversion Principle (DIP).
- Using constructor injection (passing dependencies via constructor) enables the actual usage of interfaces and change dependencies without changing code.

# How to choose the right collection in Java?
Follow the [link](https://www.baeldung.com/java-choose-list-set-queue-map) to Baeldung page about it.

# Why encapsulation?
Encapsulation (setting and getting through methods and not directly)
- Allows to intercept
- Smart getters and setters (not only sets values but does validation too). Does not comply with SOLID.
- Easier to find where the problem is if it happens when setting or getting.
- Safer in concurrency


# SOLID
## S - Single Responsibility Principle (SRP)
- A class should have only one reason to change.
- Keep responsibilities separated to make code easier to maintain and less error-prone.

## O - Open/Closed Principle (OCP)
- Classes should be open for extension but closed for modification.
- Add new functionality by adding code (e.g., new classes), not by changing existing, working code.

## L - Liskov Substitution Principle (LSP)
- Subtypes (dependencies) must be substitutable (exchangeable) for their base (interface) types without breaking the program.
- New implementations of an interface or superclass shouldn't cause unexpected behavior or errors.

## I - Interface Segregation Principle (ISP)
- Clients should not be forced to depend on interfaces they don't use.
- Prefer many small, focused interfaces rather than one large, general-purpose interface.
- Segregate (split) interfaces to ensure that classes only implement what they need.

## D - Dependency Inversion Principle (DIP)
- High-level modules should not depend on low-level modules; both should depend on abstractions (interfaces).
- Abstractions should not depend on details; details should depend on abstractions.
- Enables easy swapping of implementations and better testability.

# SOFTWARE DESIGN PATTERNS
## CREATIONAL
### SINGLETON
#### Code example
##### EAGER LOADING (Will have an instance even if never called and not needed)
This implementation is also thread-safe, because for the new instances of classes, the thread safety is managed by JDK (Java class loader)
```
class Singleton {
  private static Singleton INSTANCE = new Singleton();

  private Singleton() {}

  public static Singleton getInstance() {
    return INSTANCE;
  }
}
```
##### LAZY LOADING (Create the object only when we call the static method for the first time)
This is a more complex implementation, but could save some resources, since the instance is not created if it is not needed<br>
The `Holder` inner class is needed to ensure the thread-safety (JDK ensures thread-safety for new instances).<br> 
Since the `Holder` class is only called from inside `getInstance()`, the instance will not be created before we call `getInstance()` method for the first time.
```
class Singleton {

  private static class Holder{
    static final Singleton INSTANCE = new Singleton();
  }

  private Singleton() {}

  public static Singleton getInstance() {
    return Holder.INSTANCE;
  }
}
```

#### Summary:
- Guarantee one instance
- Easy to implement
- Solves a well defined problem
- Don't abuse it
- Don't confuse with Factory patterns

### BUILDER
#### Code example
```
// Product class
class House {
    private String foundation;
    private String walls;
    private String roof;

    // Private constructor - only Builder can create
    private House(HouseBuilder builder) {
        this.foundation = builder.foundation;
        this.walls = builder.walls;
        this.roof = builder.roof;
    }

    public static HouseBuilder builder() {
        return new HouseBuilder();
    }

    // Builder Class
    private static class HouseBuilder {
        private String foundation;
        private String walls;
        private String roof;

        public HouseBuilder foundation(String foundation) {
            this.foundation = foundation;
            return this;
        }

        public HouseBuilder walls(String walls) {
            this.walls = walls;
            return this;
        }

        public HouseBuilder roof(String roof) {
            this.roof = roof;
            return this;
        }

        public House build() {
            return new House(this);
        }
    }
}

// Usage
public class BuilderExample {
    public static void main(String[] args) {
        House house = House.builder()
                .foundation("Concrete foundation")
                .walls("Brick walls")
                .roof("Tiled roof")
                .build();

        System.out.println(house);
    }
}
```

#### Summary:
- Avoids multiple constructors
- Few drawbacks
- Immutable (only use when the objects must be immutable)

### PROTOTYPE
#### Code example
```
public class Registry {

	private final Map<String, Item> items = new HashMap<>();
	
	public Registry() {
		loadItems();
	}
	
	public Item createItem (String type) {
		Item item = null;

		try {
			item = (Item)(items.get(type)).clone();
		} catch (CloneNotSupportedException e) {
			e.printStackTrace();
		}

		return item;
	}
	
	private void loadItems() {
		Movie movie = new Movie();
		movie.setTitle("Basic Movie");
		movie.setPrice(24.99);
		movie.setRuntime("2 hours");
		items.put("Movie", movie);
		
		Book book = new Book();
		book.setNumberOfPages(335);
		book.setPrice(19.99);
		book.setTitle("Basic Book");
		items.put("Book", book);
	}
}

public abstract class Item implements Cloneable {
	private String title;
	private double price;
	private String url;
	
	@Override
	protected Object clone() throws CloneNotSupportedException {
		return super.clone();
	}
	
	public String getTitle() {
		return title;
	}
	public void setTitle(String title) {
		this.title = title;
	}
	public double getPrice() {
		return price;
	}
	public void setPrice(double price) {
		this.price = price;
	}
	public String getUrl() {
		return url;
	}
	public void setUrl(String url) {
		this.url = url;
	}
	
}

public class Movie extends Item {

	private String runtime;

	public String getRuntime() {
		return runtime;
	}

	public void setRuntime(String runtime) {
		this.runtime = runtime;
	}
	
}

public class PrototypeDemo {

	public static void main(String[] args) {
		Registry registry = new Registry();
		Movie movie = (Movie) registry.createItem("Movie");
		movie.setTitle("Creational Patterns in Java");
		
		System.out.println(movie);
		System.out.println(movie.getRuntime());
		System.out.println(movie.getTitle());
		System.out.println(movie.getUrl());
		
		Movie anotherMovie = (Movie) registry.createItem("Movie");
		anotherMovie.setTitle("Gang of Four");
		
		System.out.println(anotherMovie);
		System.out.println(anotherMovie.getRuntime());
		System.out.println(anotherMovie.getTitle());
		System.out.println(anotherMovie.getUrl());
	}

}
```

#### Shallow and Deep copies
##### Shallow copy:

- The object itself is copied, but its references (like fields that point to other objects) are not.
- Imagine copying a library card catalog: you duplicate the card entries, but the cards still point to the same books on the shelf.
- This means if you change a referenced object in one copy, the change shows up in the other, since they share that object.

##### Deep copy:

- The object and all the objects it refers to are copied recursively.
- Back to our library example: not only are the cards copied, but also new books are created for the new catalog.
- That way, changes in one copy do not affect the other.

#### Summary:
- Use to avoid costly object creation (Database connection, threads, parsing, serialization, GUI)
- DTOs are not costly objects
- Helps to avoid the use of the keyword `new`
- Utilizes `Cloneable` interface
- Usually a `Registry` is required to keep track of all the `Prototype Objects`
- Usually used when refactoring to deal with performance issues
- Also known as just a factory pattern

### FACTORY METHOD
#### Code example
```
public abstract class Website {

    public List<Page> getPages() {
        return pages;
    }

    protected List<Page> pages = new ArrayList<>();

    public Website() {
        this.createWebsite();
    }

    public abstract void createWebsite();
}

public class Shop extends Website {

    @Override
    public void createWebsite() {
        pages.add(new CartPage());
        pages.add(new ItemPage());
        pages.add(new SearchPage());
    }
}

public class Blog extends Website {

    @Override
    public void createWebsite() {
        pages.add(new PostPage());
        pages.add(new AboutPage());
        pages.add(new CommentPage());
        pages.add(new ContactPage());
    }
}

public enum WebsiteType {
    BLOG,SHOP;
}

public class WebsiteFactory {

    public static Website getWebsite(WebsiteType siteType) {

        switch (siteType) {
            case BLOG: {
                return new Blog();
            }
            case SHOP: {
                return new Shop();
            }
            default:{
                return null;
            }
        }
    }
}

public class FactoryDemo {

    public static void main (String[] args) {
        Website site = WebsiteFactory.getWebsite(WebsiteType.BLOG);

        System.out.println(site.getPages());

        site = WebsiteFactory.getWebsite(WebsiteType.SHOP);

        System.out.println(site.getPages());
    }
}
```

#### Summary:
- Creation of the factory methods take place in the concrete classes themselves by overriding the factory method defined in the contract
- Implementation might be complex
- Usually not introduced while refactoring and needs to be designed from the beginning
- Contract (interface or abstract class) driven
- Parameter driven (parameter specifies which subclass object the factory should return)
- Lets you choose a type of object to construct at runtime

### ABSTRACT FACTORY
#### Code example
```
public enum CardType {
	GOLD, PLATINUM;
}

public class AmexPlatinumCreditCard extends CreditCard {

}
public class AmexGoldCreditCard extends CreditCard {

}

public class VisaPlatinumCreditCard extends CreditCard {

}
public class VisaGoldCreditCard extends CreditCard {

}

//AbstractProduct
public abstract class CreditCard {

	protected int cardNumberLength;
	
	protected int cscNumber;

	public int getCardNumberLength() {
		return cardNumberLength;
	}

	public void setCardNumberLength(int cardNumberLength) {
		this.cardNumberLength = cardNumberLength;
	}

	public int getCscNumber() {
		return cscNumber;
	}

	public void setCscNumber(int cscNumber) {
		this.cscNumber = cscNumber;
	}
	
}

public class VisaFactory extends CreditCardFactory {

	@Override
	public CreditCard getCreditCard(CardType cardType) {
        return switch (cardType) {
            case GOLD -> new VisaGoldCreditCard();
            case PLATINUM -> new VisaPlatinumCreditCard();
        };

    }

}

public class AmexFactory extends CreditCardFactory {

	@Override
	public CreditCard getCreditCard(CardType cardType) {

        return switch (cardType) {
            case GOLD -> new AmexGoldCreditCard();
            case PLATINUM -> new AmexPlatinumCreditCard();
            default -> throw new IllegalStateException("Invalid cardType: " + cardType);
        };
	}
}

//AbstractFactory
public abstract class CreditCardFactory {

	public static CreditCardFactory getCreditCardFactory(int creditScore) {
		if(creditScore > 650) {
			return new AmexFactory();
		}
		else {
			return new VisaFactory();
		}
	}

	public abstract CreditCard getCreditCard(CardType cardType);

}

public class AbstractFactoryDemo {

	public static void main(String[] args) {
		
		CreditCardFactory abstractFactory = CreditCardFactory.getCreditCardFactory(775);
		
		CreditCard card = abstractFactory.getCreditCard(CardType.PLATINUM);
		
		System.out.println(card.getClass());
		
		abstractFactory = CreditCardFactory.getCreditCardFactory(600);
		
		CreditCard card2 = abstractFactory.getCreditCard(CardType.GOLD);
		
		System.out.println(card2.getClass());
	}

}
```

#### Summary:
- Factory of factories (factory methods)
- Most complex creational pattern
- Pattern that contains other pattern
- Problem specific (grouping of factories)
- Often starts with `Factory Method` and after is refactored into `Abstract Factory`
- Heavy abstraction
- If you only need one kind of object → use `Factory Method`.
- If you need a group of related objects that must match each other → use `Abstract Factory`.

## STRUCTURAL
### ADAPTER
#### Code example
```
public class EmployeeAdapterCSV implements Employee {

	private EmployeeCSV instance;

	public EmployeeAdapterCSV(EmployeeCSV instance) {
		this.instance = instance;
	}

	@Override
	public String getId() {
		return instance.getId() + "";
	}

	@Override
	public String getFirstName() {
		return instance.getFirstname();
	}

	@Override
	public String getLastName() {
		return instance.getLastname();
	}

	@Override
	public String getEmail() {
		return instance.getEmailAddress();
	}

}

public class EmployeeCSV {

	private int id;
	private String firstname;
	private String lastname;
	private String emailAddress;

	public EmployeeCSV(String values) {
		StringTokenizer tokenizer = new StringTokenizer(values, ",");
		if (tokenizer.hasMoreElements()) {
			id = Integer.parseInt(tokenizer.nextToken());
		}
		if (tokenizer.hasMoreElements()) {
			firstname = tokenizer.nextToken();
		}
		if (tokenizer.hasMoreElements()) {
			lastname = tokenizer.nextToken();
		}
		if (tokenizer.hasMoreElements()) {
			emailAddress = tokenizer.nextToken();
		}
	}

	public String getEmailAddress() {
		return emailAddress;
	}

	public String getFirstname() {
		return firstname;
	}

	public int getId() {
		return id;
	}

	public String getLastname() {
		return lastname;
	}

	public void setEmailAddress(String emailAddress) {
		this.emailAddress = emailAddress;
	}

	public void setFirstname(String firstname) {
		this.firstname = firstname;
	}

	public void setId(int id) {
		this.id = id;
	}

	public void setLastname(String lastname) {
		this.lastname = lastname;
	}

}

public class EmployeeAdapterLdap implements Employee {

	private EmployeeLdap instance;
	
	public EmployeeAdapterLdap(EmployeeLdap instance) {
		this.instance = instance;
	}
	
	@Override
	public String getId() {
		return instance.getCn();
	}

	@Override
	public String getFirstName() {
		return instance.getGivenName();
	}

	@Override
	public String getLastName() {
		return instance.getSurname();
	}

	@Override
	public String getEmail() {
		return instance.getMail();
	}

	public String toString() {
		return "ID: " + instance.getCn();
	}
	
}

public class EmployeeLdap {

	private String cn;
	private String surname;
	private String givenName;
	private String mail;
	
	public EmployeeLdap(String cn, String surname, String givenName, String mail) {
		this.cn = cn;
		this.surname = surname;
		this.givenName = givenName;
		this.mail = mail;
	}
	
	public String getCn() {
		return cn;
	}
	public void setCn(String cn) {
		this.cn = cn;
	}
	public String getSurname() {
		return surname;
	}
	public void setSurname(String surname) {
		this.surname = surname;
	}
	public String getGivenName() {
		return givenName;
	}
	public void setGivenName(String givenName) {
		this.givenName = givenName;
	}
	public String getMail() {
		return mail;
	}
	public void setMail(String mail) {
		this.mail = mail;
	}	
}

public class EmployeeDB implements Employee {

	private String id;
	private String firstName;
	private String lastName;
	private String email;
	
	public EmployeeDB(String id, String firstName, String lastName, String email) {
		this.id = id;
		this.firstName = firstName;
		this.lastName = lastName;
		this.email = email;
	}
	
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getFirstName() {
		return firstName;
	}
	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}
	public String getLastName() {
		return lastName;
	}
	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	
	public String toString() {
		return "ID: " + id + ", First name: " + firstName + ", Last name: " + lastName + ", Email: " + email;
	}

}

public interface Employee {

	public String getId();
	public String getFirstName();
	public String getLastName();
	public String getEmail();
	
}

public class EmployeeClient {

	public List<Employee> getEmployeeList() {
	
		List<Employee> employees = new ArrayList<>();
		
		Employee employeeFromDB = new EmployeeDB("1234", "John", "Wick", "john@wick.com");
		
		employees.add(employeeFromDB);
		
		//Will not work! This is where the adapter comes into play!
		
		//Employee employeeFromLdap = new EmployeeLdap("chewie", "Solo", "Han", "han@solo.com");
		
		EmployeeLdap employeeFromLdap = new EmployeeLdap("chewie", "Solo", "Han", "han@solo.com");
		
		employees.add(new EmployeeAdapterLdap(employeeFromLdap));
		
		EmployeeCSV employeeFromCSV = new EmployeeCSV("567,Sherlock,Holmes,sherlock@holmes.com");
		
		employees.add(new EmployeeAdapterCSV(employeeFromCSV));
		
		return employees;
		
	}
	
}

public class AdapterDemo {

	public static void main(String[] args) {
		EmployeeClient client = new EmployeeClient();
		
		List<Employee> employees = client.getEmployeeList();
		
		System.out.println(employees);
	}
}
```

#### Summary:
- No additional functionality / methods (becomes `Decorator`)
- Often used to integrate with legacy code
- Simple solution
- Can provide multiple adaptors

### BRIDGE
#### Code example
In this example `Printer` is bridged with `Formatter`. This bridge gives us flexibility to add a new `Printer` without impacting the `Formatter` or adding a new `Formatter` without impact to the `Printer`.<br>
Here a new `Formatter` is introduced (`HtmlFormatter`) and this new formatter can be used with already existing `Printer` instance without any conflicts.
```
public class MoviePrinter extends Printer {

	private Movie movie;
	
	public MoviePrinter(Movie movie) {
		this.movie = movie;
	}
	
	@Override
	protected List<Detail> getDetails() {
		List<Detail> details = new ArrayList<>();
		
		details.add(new Detail("Title", movie.getTitle()));
		details.add(new Detail("Year", movie.getYear()));
		details.add(new Detail("Runtime", movie.getRuntime()));

		return details;
	}

	@Override
	protected String getHeader() {
		return movie.getClassification();
	}

}

public abstract class Printer {

	public String print(Formatter formatter) {
		return formatter.format(getHeader(), getDetails());
	}
	
	abstract protected List<Detail> getDetails();
	
	abstract protected String getHeader();
}

public class HtmlFormatter implements Formatter {

	@Override
	public String format(String header, List<Detail> details) {
		StringBuilder builder = new StringBuilder();
		builder.append("<table>");
		builder.append("<th>");
		builder.append("Classification");
		builder.append("</th>");
		builder.append("<th>");
		builder.append(header);
		builder.append("</th>");

		for (Detail detail : details) {
			builder.append("<tr><td>");
			builder.append(detail.label());
			builder.append("</td><td>");
			builder.append(detail.value());
			builder.append("</td></tr>");
		}
		builder.append("</table>");

		return builder.toString();
	}

}

public class PrintFormatter implements Formatter {

	@Override
	public String format(String header, List<Detail> details) {
		StringBuilder builder = new StringBuilder();
		builder.append(header);
		builder.append("\n");

		for (Detail detail : details) {
			builder.append(detail.getLabel());
			builder.append(":");
			builder.append(detail.getValue());
			builder.append("\n");
		}

		return builder.toString();
	}
}

public interface Formatter {
	String format(String header, List<Detail> details);
}

public record Detail(String label, String value) {}

public class Movie {

	private String classification;
	private String runtime;
	private String title;
	private String year;

	public String getClassification() {
		return classification;
	}
	public String getRuntime() {
		return runtime;
	}

	public String getTitle() {
		return title;
	}

	public String getYear() {
		return year;
	}

	public void setClassification(String classification) {
		this.classification = classification;
	}

	public void setRuntime(String runtime) {
		this.runtime = runtime;
	}

	public void setTitle(String title) {
		this.title = title;
	}

	public void setYear(String year) {
		this.year = year;
	}
}

public class BridgeDemo {

	public static void main(String[] args) {
		Movie movie = new Movie();
		movie.setClassification("Action");
		movie.setTitle("John Wick");
		movie.setRuntime("2:15");
		movie.setYear("2014");
		
		Formatter printFormatter = new PrintFormatter();
		Printer moviePrinter = new MoviePrinter(movie);
		
		String printedMaterial = moviePrinter.print(printFormatter);
		
		System.out.println(printedMaterial);
		
		Formatter htmlFormatter = new HtmlFormatter();
		
		String htmlMaterial = moviePrinter.print(htmlFormatter);
		
		System.out.println(htmlMaterial);
	}

}
```

#### Summary:
- Decouples contract from the implementation (interface from the class)
- Allows to make changes to the contracts without breaking the implementations
- Increases complexity
- Conceptually difficult to plan
- Designed in advance
- Provides flexibility with a cost of complexity

# Kubernetes
Kubernetes helps us manage a lot of containers

#### Main benefits
- Speed of deployment
- Ability to absorb change quickly
- Ability to recover quickly
- Hide complexity in the cluster

#### Infrastructure Abstraction
One of the main features that kubernetes provides is **Infrastructure Abstraction**
Which means when deploying developers don't need to know or care about specific nodes or do any manual jobs to make sure the load balancer sends traffic to the application, it is all done under the hood

#### Desired State
We can describe how our application structure looks like (all different services/controllers) and Kubernetes ensures it maintains this **desired state** (all the services are up, how they interact with each other, etc.)

#### Kubernetes API
Provides a way to deploy and manage the Kubernetes services and also this is where Kubernetes Cluster components interact with each other

## Kubernetes API Objects
Kubernetes API Objects are the components which help us configure and deploy applications to Kubernetes. Using these objects we define the state and structure of the system. We can do that both **Declaratively** (using .yml) and **Imperatively** (setting everything up using the command line)

### Pods
- One or more containers
- Most basic unit of work. The **single** application / service.
- Amount of resources needed can be described and kubernetes makes sure those resources are available to the **Pod**
- **Ephemeral** (no **Pod** is ever "redeployed" if **Pod** fails, new one will replace it)
- **Atomicity** - the **Pod** is either there or NOT (**Pod** **cannot be partially running**). This makes more sense when more than 1 container is running on the **Pod**. If at least 1 of the containers fail, **Pod** is no longer available.
- Using **Probes**, the health of the application inside a **Pod** can be checked. Even though the **Pod** is running, the inside application might be throwing errors and using **Probes** we can ensure that doesn't happen without us knowing.

### Controllers
Makes sure our Pods are running and healthy.
- Defines the desired state
- Can create and manage Pods for you
- Monitors and responds to Pod state and health

Types of controllers:
- *ReplicaSet.* - allows to define a number of replicas for a pod and makes sure all the replicas run and are healthy
- *Deployment* - ReplicaSets don't usually get created by themselves, they are defined in the *Deployment* controllers and are created by them. *Deployment* controller also manages the transition between 2 ReplicaSets (for example moving between 2 versions of an application)

### Services
Helps to keep Pods **persistent** (since they are *ephemeral*)
- Service is provided with an **IP** address and a **DNS** name
- Users can just simply reach new pods by communicating with the Service instead of individual pods, that get deleted and recreated
- Helps to manage easy **Scaling**
- Handles all the **routing** to pods
- Provides **Load Balancing**

### Storage
**Persistent Volume Claim** is used to reserve some *Cluster* storage for each Pod. Each pod uses their storage independently

### ConfigMap
ConfigMap is an API object used to store non-confidential data in key-value pairs

Example: 
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true  
```

## Cluster components

### Control Plane Node (formerly Master Node)
Implements main control functions of a cluster, does monitoring and scheduling, coordinates cluster operations

Contains these components:
- #### API Server
    Communications hub for the cluster<br>
    **kubectl** is used to interact with the **API Server**<br>
    Uses RESTful principle
- #### etcd
    Cluster datastore. Persists the **state** of the kubernetes **API Objects** using **key-value pairs**
- #### Scheduler
    Tells kubernetes on which nodes the Pod should be started based on the Pods **resource** requirements and other Pod requirements<br>
    **Watches** the **API server** and looks for **unscheduled Pods** and schedules them onto Nodes in the Cluster<br>
    **Constraints** can be defined (2 specific Pods to never be on the same Node, etc.) and **Scheduler** follows them
- #### Controller manager
    Responsible for keeping things in the desired state. Implements lifecycle functions for controllers that monitor and manage kubernetes objects state.

### Node
The worker node is where our application pods run. Starts pods and their containers 

Each node has these services:
#### Kubelet
- Monitors API Server for changes
- Responsible for Pod Lifecycle (starts and stops pods and their containers)
- Reports Node & Pod state
- Executes Pod *probes*

#### kube-proxy
- most commonly implemented in **iptables**
- implements Services
- Routes traffic to Pods
- Load balancing

#### Container Runtime
- Downloads images & runs containers
- uses Container Runtime Interface (CRI) which allows to choose a desired container runtime
- the default runtime since 1.20 is ***containerd*** before then it was *Docker*

### Cluster Add-on Pods
Special purpose Pods

Examples:
- #### DNS Server Pod
    Provides DNS for the Cluster
- #### Ingres controller
    Advanced load balancers

## Kubernetes Networking
- Pods on a Node can communicate to all Pods on all Nodes without Network Address Translation (NAT)
- Agents (kubelet, System Daemon, etc.) on a Node can communicate with all Pods on that Node

### CNI (Container Network Interface) can be used to satisfy Kubernetes networking requirements
Kubernetes by itself doesn’t implement networking. Instead, it defines networking requirements that can be satisfied by a Container Network Interface (CNI) plugin.
**Calico** is one of the most popular CNI implementations.

## Installing Kubernetes with **kubeadm**
### Where to install?
- Cloud
    - IaaS - virtual machines (have to take care of the OS and Kubernetes software on the virtual machines)
    - PaaS - managed service (provided by major cloud providers, don't have to maintain anything)
- On-premises (have to take care of everything)
    - Bare metal
    - Virtual machines

While Cloud takes care of the hardware and/or software maintenance, on-prem solution might be cheaper and more flexible.

### Installation Requirements
- #### System
    - Linux OS
    - 2 CPUs
    - 2GB RAM
- #### Container runtime
    - Container Runtime Interface (CRI)
    - containerd
    - CRI-O
- #### Networking
    - Connectivity between all nodes
    - Unique hostname
    - Unique MAC address

### Installation Steps
#### 1. Define your Cluster structure and network topology
For example a Cluster with a control plane node and 2 worker nodes. 

Assign an internal IP address to each of the nodes.

#### 2. Install packages on each node
- containerd (or other container runtime like Docker)
- kubelet
- kubeadm
- kubectl

See [kubernetes package installation guide](kubernetes/0-PackageInstallation-containerd.sh)

#### 3. Bootstrap a Cluster with kubeadm - Control Plane Node
kubeadm init will: 
- execute a series of pre-flight checks:
    - ensures you have right permissions on the system
    - ensures you have enough system resources
    - ensures there's a container runtime and it is up and running
- create a Certificate Authority (CA)
    - secures cluster communication
    - creates certificates for user and cluster component authentication
    - CA files are stored in /etc/kubernetes/pki
- generate kubeconfig files so cluster components could locate each other
    - defines how to connect to your cluster
        - Client certificates
        - Cluster API network location
    - stored in /etc/kubernetes
        - admin.conf (kubernetes-admin)
        - super-admin.conf
        - controller-manager.conf
        - scheduler.conf
        - kubelet.conf
- generate static pod manifests
    - manifest describes a configuration
    - lives in /etc/kubernetes/manifests
    - watched by the **kubelet** to start automatically on system start or restart
- wait for the control plane pods to start
- taint the control plane node (so user pods won't be scheduled onto control plane node)
- generate a bootstrap token (used to add nodes to the cluster)
- start add-on pods

To create the Control Plane Node follow the [link](kubernetes/1-CreateControlPlaneNode-containerd.sh)

#### 4. Bootstrap a Cluster with kubeadm - Worker Node
**kubeadm join** will be used to join a node to a cluster

kubeadm join will:
- download cluster information
- submit CSR (Certificate Signing Request) and receive a certificate that node kubelet will use to join the cluster
- CA automatically signs the CSR, kubeadm join downloads the certificate and stores it in a file system of the node (/var/lib/kubelet/pki)
- generate kubelet.conf file

Follow the [link](kubernetes/2-CreateNodes-containerd.sh) for further instructions on how to join the worker node onto the cluster

## Working with Clusters
The main tool to control a running Cluster is **kubectl**

usage: kubectl [operation] [resource] [name] [flags (e.g. output)]<br>
Note: name is optional if you want to work with multiple resources

More on usage in [kubernetes kubectl documentation](https://kubernetes.io/docs/reference/kubectl/kubectl/) and [cheat sheet](https://kubernetes.io/docs/reference/kubectl/quick-reference/)

### kubectl operations
- apply/create - create resource
- run - start a pod from an image (a pod that is not managed by a controller)
- explain - documentation of resources
- delete - delete resource
- get - list resources
- describe - detailed resource information
- exec - execute a command on a container
- logs - view logs on a container

### kubectl resources
- nodes (no)
- pods (po)
- services (svc)
- ...more resources [here](https://kubernetes.io/docs/reference/kubectl/overview/#resource-types)

### kubectl output
By adding additional flags, kubectl output can be modified. Output flag is -o
- wide - output additional information
- yaml - YAML formatted API object
- json - JSON formatted API object
- dry-run - print an object without sending it to the API Server (good for generating YAML on how to create resources)

Examples of kubectl usage and how to enable bash kublectl completion [here](kubernetes/1-workingwithyourcluster.sh)

## Application Deployment in Kubernetes
### Imperative configuration (executing commands in cli one at a time)
- kubectl create deployment nginx -- image=nginx
- kubectl run nginx -image=nginx
### Declarative (Configured in a file with desired state)
- Define desired state in code
- Manifest in YAML or JSON
- kubectl apply -f deployment.yaml

Using imperative way with a dry run flag (--dry-run=client -o yaml > deployment.yaml) could help generate correct Declarative configuration

Instructions on how to deploy your application into the cluster and how to modify existing deployments [here](kubernetes/2-deployingapplications.sh)

## Cluster Maintenance
### Cluster Upgrade Process
- Upgrade Control Plane Node -> Upgrade any other Control Plane Nodes -> Upgrade Worker Nodes
- Can only upgrade minor versions
    - 1.28 -> 1.29
    - 1.27 X 1.29
- Read the release notes before each upgrade
### Worker Node Maintenance
- Version upgrades, OS updates and hardware updates
#### Steps:
- ##### Drain/Cordon the Node
    - kubectl drain NODE_NAME
        - Marks the node unschedulable
        - Gracefully terminates Pods (Controller automatically reschedules them on other Nodes)
- ##### Keep resources in mind
    Before draining the Node be sure you have enough resources on the other Nodes to take over the work of the Node you're about to drain and upgrade

### Upgrade Control Plane Node
- Update kubeadm package (apt or yum upgrade)
- kubeadm upgrade plan
    - does pre-flight checks like ensuring you're upgrading to an appropriate version, nodes are healthy, etc.
- kubeadm upgrade apply
    - runs pre-flight checks
    - pre-pulls the container images
    - updates certificates for each control plane pod
    - creates new static pod manifests in /etc/kubernetes/manifests
    - saves previous manifests in /etc/kubernetes/temp
- drain the Control Plane Node
    - drain only non-control plane pods
    - control plane pods will stay on control plane node, since they are static
- upgrade kubelet and kubectl
- uncordon/undrain the Control Plane Node

Instructions on how to do it in terminal [here](./kubernetes/1-ControlPlaneUpgrade.sh)

### Upgrade Worker Nodes
Be sure to upgrade worker nodes to the **same versions** as the **Control Plane Nodes**
- update kubeadm
- kubeadm upgrade node
- drain the node
- update kubelet and kubectl
- uncordon the node

Instructions on how to do it in terminal [here](./kubernetes/2-NodeUpgrade.sh)


## Probes
### Liveness probes
Used by kubelet to know when to restart a container
### Readiness probes
Used by kubelet to know when the container is ready to start accepting traffic. A pod is considered ready when all of its containers are ready. This is used for services. When a pod is not ready, it is removed from service load balancers.
### Startup probes
Used by kubelet to know when a container application has started. This can be used to adopt liveness checks on slow starting containers, and will avoid them getting killed by the kubelet before they are up and running.

## Init Containers
These containers are used to support the main containers of the pod and are run before app containers start. They are mainly used for setup scripts.

## Kustomize
Community-led solution for managing **Kubernetes configuration**. Allows to take configuration expressed as raw YAML and customize the YAML for our purposes. It leaves the **original YAML** definitions **untouched**. A **good use case** is to use **Kustomize** to apply **DRY principle** while writing Kubernetes configuration for different environments (dev, stage, prod). Meaning to not rewrite configurations for each environment and to reuse the same one, but add minor changes related to that environment.

Uses 2 broad classes to achieve the result:
- Generators - create new Kubernetes objects when they don't exist and transform existing object when they do exist
- Transformers - take existing Kubernetes objects and overlay alternative values and fields to make different variations of the configuration. 

Kustomize key facts:
- Originates from Google. It's a sub-project of the Kubernetes community CLI SIG (SIG-CLI)
- Kustomize is both, a standalone binary, and also an integral part of Kubernetes CLI (kubectl)
- Kustomize has many features, but if a required feature is missing this can be provided using a plugin.


### Kubernetes Resource Model (KRM)
A declarative way to express Kubernetes clusters. Kustomize adopts the familiar KRM approach for defining operations on configuration.

The configuration is described in `kustomization.yaml` file.<br>
Example:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```

### Built-in Generators
- ConfigMap - Creates, replaces or merges one or more ConfigMap objects
- Secret - Creates, replaces or merges one or more Secret objects

### Built-in Transformers
- Namespace - Sets namespace for some or all objects
- ImageTag - Sets image name, tag or digest
- PrefixSuffix - Adds a prefix or suffix to a name
- ReplicaCount - Increase or decrease number of replicas

### Built-in Metadata Transformers
- Label - Adds labels to the metadata for objects and their corresponding selectors
- Annotation - Adds annotation key/value pairs to the metadata for all applicable objects

### Resources
- List of directories pointing to configuration files that are going to be extended / changed
- Web URLs can be used to provide remote resources. Web URLs are separated by // to provide a path location to resource within the remote locations (e.g. `https://git.com/myloc//path/to/deployments`)

### Setting namespaces
An example of how to set namespaces with kustomize:
```
---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../base
  # Include the namespace to add the managed-by label
  - namespace.yaml
buildMetadata:
  - managedByLabel
namespace: dev
```

namespace.yaml:
```
---
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

Namespaces can also be **changed** using NamespaceTransformer instead

### Adding labels
You can add label to all objects (resources) metadata or make **common** labels, that will be applied to not only the metadata, but the selectors and the templates
```
---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../base
  - namespace.yaml
buildMetadata:
  - managedByLabel
namespace: dev
# appears only in the metadata
labels: 
  - pairs:
      app.kubernetes.io/env: dev
# appears everywhere
commonLabels:
  app.kubernetes.io/version: v1.0

```

### Generating ConfigMap
Use `configMapGenerator` to generate a **ConfigMap**<br>
Multiple **ConfigMaps** can be generated using `configMapGenerator`
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
configMapGenerator:
  - name: db
    literals:
      - SQLITE_DB_LOCATION=/tmp/todos/todo.db
```

Similarly `Kubernetes Secretes` can be managed too:
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
secretGenerator:
  - name: registry-credentials
    files:
      - config.json
    type: "kubernetes.io/dockerconfigjson"
```
The main difference is that when specifying `Kubernetes Secrets` you can specify a **type**

### Patching
- #### Strategic Merge Patch
	- Schema aware
	- It can merge sections instead of overwriting entire sections
	- Example:
	```
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: my-app
      spec:
        template:
          spec:
            containers:
              - name: app
                image: nginx:1.19
	```
- #### JSON Patch
	- Not schema aware
	- Used when merge patch cannot express your change
    - Patch operations: Add, Remove, Replace, Move, Copy, Test<br>
    - Example:
    ```
    # deployment-patch.yaml
    - op: replace
      path: /spec/template/spec/containers/0/imagePullPolicy
      value: Always
    ```

# GitHub Actions CI/CD

Simple example:
```
name: api-build

on:
  push:
    paths:
      - ".github/workflows/api-build.yml"
      - "api/**"
  workflow_dispatch: # manual

jobs:
  build:
    name: build-dotnet-api
    runs-on: ubuntu-latest
	env:
		GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: actions/checkout@v4
	  	with:
			my-param-version: '1.9.0'
      - run: dotnet --list-runtimes
      - run: dotnet --list-sdks
      - run: dotnet build
        working-directory: ./api
      - run: dotnet test
        working-directory: ./api
```

- `name` - what you see in GitHub Actions tab workflow list on the left side
- `on` - describe when will the workflow be triggered
- `jobs` - describe the actions that will be executed when the workflow is triggered
	- `name` - describes the name of the job (for example a job can be build, another job can be test and so on)
	- `runs-on` - runner, could either be github hosted runners or self-hosted
	- `steps` - the specifics steps that will be run during that job
		- `uses` - use github action from the action marketplace
		- `run` - run a specific command
		- `working-directory` - the directory where the step is gonna run from
		- `with` - provides a parameter for the step (some steps will have required parameters)
	- `env` - provide environment variables [list of default github actions env variables](https://docs.github.com/en/actions/reference/workflows-and-actions/variables)
- `needs` - can provide another job that should run before this