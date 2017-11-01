---
title: hibernate表关系映射示例
date: 2017-11-01 17:18:44
categories: jpa/hibernate
tags: hibernate-relation
---

## 1. OneToOne
一对一关系，存在n+1问题。

#### 1.1 单向 @OneToOne
- 实体：


        @Entity(name = "Phone")
        public static class Phone {
        
            @Id
            @GeneratedValue
            private Long id;
        
            @Column(name = "`number`")
            private String number;
        
            @OneToOne
            @JoinColumn(name = "details_id")
            private PhoneDetails details;
        
            public Phone() {
            }
        
            public Phone(String number) {
                this.number = number;
            }
        
            public Long getId() {
                return id;
            }
        
            public String getNumber() {
                return number;
            }
        
            public PhoneDetails getDetails() {
                return details;
            }
        
            public void setDetails(PhoneDetails details) {
                this.details = details;
            }
        }
	
---	
        @Entity(name = "PhoneDetails")
        public static class PhoneDetails {
        
            @Id
            @GeneratedValue
            private Long id;
        
            private String provider;
        
            private String technology;
        
            public PhoneDetails() {
            }
        
            public PhoneDetails(String provider, String technology) {
                this.provider = provider;
                this.technology = technology;
            }
        
            public String getProvider() {
                return provider;
            }
        
            public String getTechnology() {
                return technology;
            }
        
            public void setTechnology(String technology) {
                this.technology = technology;
            }
        }
	
- *生成sql*：

        CREATE TABLE Phone (
            id BIGINT NOT NULL ,
            number VARCHAR(255) ,
            details_id BIGINT ,
            PRIMARY KEY ( id )
        )
        
        CREATE TABLE PhoneDetails (
            id BIGINT NOT NULL ,
            provider VARCHAR(255) ,
            technology VARCHAR(255) ,
            PRIMARY KEY ( id )
        )
        
        ALTER TABLE Phone
        ADD CONSTRAINT FKnoj7cj83ppfqbnvqqa5kolub7
        FOREIGN KEY (details_id) REFERENCES PhoneDetails
	
	
- *操作*：
	作为外键，相当于ManyToOne作为外键操作

---
	
#### 1.2 双向 @OneToOne

- 实体：


    @Entity(name = "Phone")
    public static class Phone {
    
        @Id
        @GeneratedValue
        private Long id;
    
        @Column(name = "`number`")
        private String number;
    
        @OneToOne(mappedBy = "phone", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
        private PhoneDetails details;
    
        public Phone() {
        }
    
        public Phone(String number) {
            this.number = number;
        }
    
        public Long getId() {
            return id;
        }
    
        public String getNumber() {
            return number;
        }
    
        public PhoneDetails getDetails() {
            return details;
        }
    
        public void addDetails(PhoneDetails details) {
            details.setPhone( this );
            this.details = details;
        }
    
        public void removeDetails() {
            if ( details != null ) {
                details.setPhone( null );
                this.details = null;
            }
        }
    }
	
---
	
	@Entity(name = "PhoneDetails")
	public static class PhoneDetails {
	
	    @Id
	    @GeneratedValue
	    private Long id;
	
	    private String provider;
	
	    private String technology;
	
	    @OneToOne(fetch = FetchType.LAZY)
	    @JoinColumn(name = "phone_id")
	    private Phone phone;
	
	    public PhoneDetails() {
	    }
	
	    public PhoneDetails(String provider, String technology) {
	        this.provider = provider;
	        this.technology = technology;
	    }
	
	    public String getProvider() {
	        return provider;
	    }
	
	    public String getTechnology() {
	        return technology;
	    }
	
	    public void setTechnology(String technology) {
	        this.technology = technology;
	    }
	
	    public Phone getPhone() {
	        return phone;
	    }
	
	    public void setPhone(Phone phone) {
	        this.phone = phone;
	    }
	}

- *生成sql*：


    CREATE TABLE Phone (
        id BIGINT NOT NULL ,
        number VARCHAR(255) ,
        PRIMARY KEY ( id )
    )
    
    CREATE TABLE PhoneDetails (
        id BIGINT NOT NULL ,
        provider VARCHAR(255) ,
        technology VARCHAR(255) ,
        phone_id BIGINT ,
        PRIMARY KEY ( id )
    )
    
    ALTER TABLE PhoneDetails
    ADD CONSTRAINT FKeotuev8ja8v0sdh29dynqj05p
    FOREIGN KEY (phone_id) REFERENCES Phone

- *操作*：


	Phone phone = new Phone( "123-456-7890" );
	PhoneDetails details = new PhoneDetails( "T-Mobile", "GSM" );
	
	phone.addDetails( details );
	entityManager.persist( phone );
	
	-------------------------------------
	INSERT INTO Phone ( number, id )
	VALUES ( '123 - 456 - 7890', 1 )
	
	INSERT INTO PhoneDetails ( phone_id, provider, technology, id )
	VALUES ( 1, 'T - Mobile, GSM', 2 )
	

---
## 2. OneToMany
一对多关系，一般在多的一段维护，也可双边维护关系。

#### 2.1 单向 @OneToMany association

- 实体:


    @Entity(name = "Person")
    public static class Person {
    
        @Id
        @GeneratedValue
        private Long id;
        @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
        private List<Phone> phones = new ArrayList<>();
    
        public Person() {
        }
    
        public List<Phone> getPhones() {
            return phones;
        }
    }
	
---	
	
	@Entity(name = "Phone")
	public static class Phone {
	
	    @Id
	    @GeneratedValue
	    private Long id;
	
	    @Column(name = "`number`")
	    private String number;
	
	    public Phone() {
	    }
	
	    public Phone(String number) {
	        this.number = number;
	    }
	
	    public Long getId() {
	        return id;
	    }
	
	    public String getNumber() {
	        return number;
	    }
	}
	
- *sql生成*：


	CREATE TABLE Person (
	    id BIGINT NOT NULL ,
	    PRIMARY KEY ( id )
	)

	CREATE TABLE Person_Phone (
	    Person_id BIGINT NOT NULL ,
	    phones_id BIGINT NOT NULL
	)
	
	CREATE TABLE Phone (
	    id BIGINT NOT NULL ,
	    number VARCHAR(255) ,
	    PRIMARY KEY ( id )
	)
	
	ALTER TABLE Person_Phone
	ADD CONSTRAINT UK_9uhc5itwc9h5gcng944pcaslf
	UNIQUE (phones_id)
	
	ALTER TABLE Person_Phone
	ADD CONSTRAINT FKr38us2n8g5p9rj0b494sd3391
	FOREIGN KEY (phones_id) REFERENCES Phone
	
	ALTER TABLE Person_Phone
	ADD CONSTRAINT FK2ex4e4p7w1cj310kg2woisjl2
	FOREIGN KEY (Person_id) REFERENCES Person

- *操作代码*：


	Person person = new Person();
	Phone phone1 = new Phone( "123-456-7890" );
	Phone phone2 = new Phone( "321-654-0987" );
	
	person.getPhones().add( phone1 );
	person.getPhones().add( phone2 );
	entityManager.persist( person );
	entityManager.flush();
	
	person.getPhones().remove( phone1 );
	
	-----------------------------------------------
	INSERT INTO Person
       ( id )
	VALUES ( 1 )
	
	INSERT INTO Phone
	       ( number, id )
	VALUES ( '123 - 456 - 7890', 2 )
	
	INSERT INTO Phone
	       ( number, id )
	VALUES ( '321 - 654 - 0987', 3 )
	
	INSERT INTO Person_Phone
	       ( Person_id, phones_id )
	VALUES ( 1, 2 )
	
	INSERT INTO Person_Phone
	       ( Person_id, phones_id )
	VALUES ( 1, 3 )
	
	DELETE FROM Person_Phone
	WHERE  Person_id = 1
	
	INSERT INTO Person_Phone
	       ( Person_id, phones_id )
	VALUES ( 1, 3 )
	
	DELETE FROM Phone
	WHERE  id = 2
	
#### 2.2 双向@OneToMany

- 实体：


	@Entity(name = "Person")
	public static class Person {
	
	    @Id
	    @GeneratedValue
	    private Long id;
	    @OneToMany(mappedBy = "person", cascade = CascadeType.ALL, orphanRemoval = true)
	    private List<Phone> phones = new ArrayList<>();
	
	    public Person() {
	    }
	
	    public Person(Long id) {
	        this.id = id;
	    }
	
	    public List<Phone> getPhones() {
	        return phones;
	    }
	
	    public void addPhone(Phone phone) {
	        phones.add( phone );
	        phone.setPerson( this );
	    }
	
	    public void removePhone(Phone phone) {
	        phones.remove( phone );
	        phone.setPerson( null );
	    }
	}		
	
---	
	
	@Entity(name = "Phone")
	public static class Phone {
	
	    @Id
	    @GeneratedValue
	    private Long id;
	
	    @NaturalId
	    @Column(name = "`number`", unique = true)
	    private String number;
	
	    @ManyToOne
	    private Person person;
	
	    public Phone() {
	    }
	
	    public Phone(String number) {
	        this.number = number;
	    }
	
	    public Long getId() {
	        return id;
	    }
	
	    public String getNumber() {
	        return number;
	    }
	
	    public Person getPerson() {
	        return person;
	    }
	
	    public void setPerson(Person person) {
	        this.person = person;
	    }
	
	    @Override
	    public boolean equals(Object o) {
	        if ( this == o ) {
	            return true;
	        }
	        if ( o == null || getClass() != o.getClass() ) {
	            return false;
	        }
	        Phone phone = (Phone) o;
	        return Objects.equals( number, phone.number );
	    }
	
	    @Override
	    public int hashCode() {
	        return Objects.hash( number );
	    }
	}
	
- *生成sql*：


	CREATE TABLE Person (
	    id BIGINT NOT NULL ,
	    PRIMARY KEY ( id )
	)
	
	CREATE TABLE Phone (
	    id BIGINT NOT NULL ,
	    number VARCHAR(255) ,
	    person_id BIGINT ,
	    PRIMARY KEY ( id )
	)
	
	ALTER TABLE Phone
	ADD CONSTRAINT UK_l329ab0g4c1t78onljnxmbnp6
	UNIQUE (number)
	
	ALTER TABLE Phone
	ADD CONSTRAINT FKmw13yfsjypiiq0i1osdkaeqpg
	FOREIGN KEY (person_id) REFERENCES Person

- *操作*：


	Person person = new Person();
	Phone phone1 = new Phone( "123-456-7890" );
	Phone phone2 = new Phone( "321-654-0987" );
	
	person.addPhone( phone1 );
	person.addPhone( phone2 );
	entityManager.persist( person );
	entityManager.flush();
	
	person.removePhone( phone1 );	
	
	-----------------------------------------
	
	INSERT INTO Phone
       ( number, person_id, id )
	VALUES ( '123-456-7890', NULL, 2 )
	
	INSERT INTO Phone
	       ( number, person_id, id )
	VALUES ( '321-654-0987', NULL, 3 )
	
	DELETE FROM Phone
	WHERE  id = 2
	
---
## 3. ManyToOne
多对一关系

#### 3.1 @ManyToOne association
相当于外键

- 实体


	@Entity(name = "Person")
	public static class Person {
	
		@Id
		@GeneratedValue
		private Long id;
		
		public Person() {
		
		}
	}
	
---	
	
	@Entity(name = "Phone")
	public static class Phone {
	
	    @Id
	    @GeneratedValue
	    private Long id;
	
	    @Column(name = "`number`")
	    private String number;
	
	    @ManyToOne
	    @JoinColumn(name = "person_id",
	            foreignKey = @ForeignKey(name = "PERSON_ID_FK")
	    )
	    private Person person;
	
	    public Phone() {
	    }
	
	    public Phone(String number) {
	        this.number = number;
	    }
	
	    public Long getId() {
	        return id;
	    }
	
	    public String getNumber() {
	        return number;
	    }
	
	    public Person getPerson() {
	        return person;
	    }
	
	    public void setPerson(Person person) {
	        this.person = person;
	    }
	}
	
- *sql生成*：


	CREATE TABLE Person (
	    id BIGINT NOT NULL ,
	    PRIMARY KEY ( id )
	)
	
	CREATE TABLE Phone (
	    id BIGINT NOT NULL ,
	    number VARCHAR(255) ,
	    person_id BIGINT ,
	    PRIMARY KEY ( id )
	)
	
	ALTER TABLE Phone
	ADD CONSTRAINT PERSON_ID_FK
	FOREIGN KEY (person_id) REFERENCES Person
	
- *生命周期*:


	Person person = new Person();
	entityManager.persist( person );
	
	Phone phone = new Phone( "123-456-7890" );
	phone.setPerson( person );
	entityManager.persist( phone );
	
	entityManager.flush();
	phone.setPerson( null );
	
	实际sql：
	INSERT INTO Person ( id )
	VALUES ( 1 )
	
	INSERT INTO Phone ( number, person_id, id )
	VALUES ( '123-456-7890', 1, 2 )
	
	UPDATE Phone
	SET    number = '123-456-7890',
	       person_id = NULL
	WHERE  id = 2

---	
## 4. ManyToMany
多对多关系，两边都要维护。

#### 4.1 单向 @ManyToMany

- 实体：


	@Entity(name = "Person")
	public static class Person {
	
	    @Id
	    @GeneratedValue
	    private Long id;
	    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
	    private List<Address> addresses = new ArrayList<>();
	
	    public Person() {
	    }
	
	    public List<Address> getAddresses() {
	        return addresses;
	    }
	}
	
---	
	@Entity(name = "Address")
	public static class Address {
	
	    @Id
	    @GeneratedValue
	    private Long id;
	
	    private String street;
	
	    @Column(name = "`number`")
	    private String number;
	
	    public Address() {
	    }
	
	    public Address(String street, String number) {
	        this.street = street;
	        this.number = number;
	    }
	
	    public Long getId() {
	        return id;
	    }
	
	    public String getStreet() {
	        return street;
	    }
	
	    public String getNumber() {
	        return number;
	    }
	}
	
- *生成sql*：


	CREATE TABLE Address (
	    id BIGINT NOT NULL ,
	    number VARCHAR(255) ,
	    street VARCHAR(255) ,
	    PRIMARY KEY ( id )
	)
	
	CREATE TABLE Person (
	    id BIGINT NOT NULL ,
	    PRIMARY KEY ( id )
	)
	
	CREATE TABLE Person_Address (
	    Person_id BIGINT NOT NULL ,
	    addresses_id BIGINT NOT NULL
	)
	
	ALTER TABLE Person_Address
	ADD CONSTRAINT FKm7j0bnabh2yr0pe99il1d066u
	FOREIGN KEY (addresses_id) REFERENCES Address
	
	ALTER TABLE Person_Address
	ADD CONSTRAINT FKba7rc9qe2vh44u93u0p2auwti
	FOREIGN KEY (Person_id) REFERENCES Person
	
- *操作*：


	Person person1 = new Person();
	Person person2 = new Person();
	
	Address address1 = new Address( "12th Avenue", "12A" );
	Address address2 = new Address( "18th Avenue", "18B" );
	
	person1.getAddresses().add( address1 );
	person1.getAddresses().add( address2 );
	
	person2.getAddresses().add( address1 );
	
	entityManager.persist( person1 );
	entityManager.persist( person2 );
	
	entityManager.flush();
	
	person1.getAddresses().remove( address1 );
	
	-------------------------------------------
	INSERT INTO Person ( id )
	VALUES ( 1 )
	
	INSERT INTO Address ( number, street, id )
	VALUES ( '12A', '12th Avenue', 2 )
	
	INSERT INTO Address ( number, street, id )
	VALUES ( '18B', '18th Avenue', 3 )
	
	INSERT INTO Person ( id )
	VALUES ( 4 )
	
	INSERT INTO Person_Address ( Person_id, addresses_id )
	VALUES ( 1, 2 )
	INSERT INTO Person_Address ( Person_id, addresses_id )
	VALUES ( 1, 3 )
	INSERT INTO Person_Address ( Person_id, addresses_id )
	VALUES ( 4, 2 )
	
	DELETE FROM Person_Address
	WHERE  Person_id = 1
	
	INSERT INTO Person_Address ( Person_id, addresses_id )
	VALUES ( 1, 3 )
	
	
#### 4.2 双向 @ManyToMany
- 实体：


	@Entity(name = "Person")
	public static class Person {
	
	    @Id
	    @GeneratedValue
	    private Long id;
	
	    @NaturalId
	    private String registrationNumber;
	    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
	    private List<Address> addresses = new ArrayList<>();
	
	    public Person() {
	    }
	
	    public Person(String registrationNumber) {
	        this.registrationNumber = registrationNumber;
	    }
	
	    public List<Address> getAddresses() {
	        return addresses;
	    }
	
	    public void addAddress(Address address) {
	        addresses.add( address );
	        address.getOwners().add( this );
	    }
	
	    public void removeAddress(Address address) {
	        addresses.remove( address );
	        address.getOwners().remove( this );
	    }
	
	    @Override
	    public boolean equals(Object o) {
	        if ( this == o ) {
	            return true;
	        }
	        if ( o == null || getClass() != o.getClass() ) {
	            return false;
	        }
	        Person person = (Person) o;
	        return Objects.equals( registrationNumber, person.registrationNumber );
	    }
	
	    @Override
	    public int hashCode() {
	        return Objects.hash( registrationNumber );
	    }
	}
	
---	
	@Entity(name = "Address")
	public static class Address {
	
	    @Id
	    @GeneratedValue
	    private Long id;
	
	    private String street;
	
	    @Column(name = "`number`")
	    private String number;
	
	    private String postalCode;
	
	    @ManyToMany(mappedBy = "addresses")
	    private List<Person> owners = new ArrayList<>();
	
	    public Address() {
	    }
	
	    public Address(String street, String number, String postalCode) {
	        this.street = street;
	        this.number = number;
	        this.postalCode = postalCode;
	    }
	
	    public Long getId() {
	        return id;
	    }
	
	    public String getStreet() {
	        return street;
	    }
	
	    public String getNumber() {
	        return number;
	    }
	
	    public String getPostalCode() {
	        return postalCode;
	    }
	
	    public List<Person> getOwners() {
	        return owners;
	    }
	
	    @Override
	    public boolean equals(Object o) {
	        if ( this == o ) {
	            return true;
	        }
	        if ( o == null || getClass() != o.getClass() ) {
	            return false;
	        }
	        Address address = (Address) o;
	        return Objects.equals( street, address.street ) &&
	                Objects.equals( number, address.number ) &&
	                Objects.equals( postalCode, address.postalCode );
	    }
	
	    @Override
	    public int hashCode() {
	        return Objects.hash( street, number, postalCode );
	    }
	}
	
- *生成sql*：


	CREATE TABLE Address (
	    id BIGINT NOT NULL ,
	    number VARCHAR(255) ,
	    postalCode VARCHAR(255) ,
	    street VARCHAR(255) ,
	    PRIMARY KEY ( id )
	)
	
	CREATE TABLE Person (
	    id BIGINT NOT NULL ,
	    registrationNumber VARCHAR(255) ,
	    PRIMARY KEY ( id )
	)
	
	CREATE TABLE Person_Address (
	    owners_id BIGINT NOT NULL ,
	    addresses_id BIGINT NOT NULL
	)
	
	ALTER TABLE Person
	ADD CONSTRAINT UK_23enodonj49jm8uwec4i7y37f
	UNIQUE (registrationNumber)
	
	ALTER TABLE Person_Address
	ADD CONSTRAINT FKm7j0bnabh2yr0pe99il1d066u
	FOREIGN KEY (addresses_id) REFERENCES Address
	
	ALTER TABLE Person_Address
	ADD CONSTRAINT FKbn86l24gmxdv2vmekayqcsgup
	FOREIGN KEY (owners_id) REFERENCES Person
	
- *操作*：


	Person person1 = new Person( "ABC-123" );
	Person person2 = new Person( "DEF-456" );
	
	Address address1 = new Address( "12th Avenue", "12A", "4005A" );
	Address address2 = new Address( "18th Avenue", "18B", "4007B" );
	
	person1.addAddress( address1 );
	person1.addAddress( address2 );
	
	person2.addAddress( address1 );
	
	entityManager.persist( person1 );
	entityManager.persist( person2 );
	
	entityManager.flush();
	
	person1.removeAddress( address1 );
	
	------------------------------------------
	INSERT INTO Person ( registrationNumber, id )
	VALUES ( 'ABC-123', 1 )
	
	INSERT INTO Address ( number, postalCode, street, id )
	VALUES ( '12A', '4005A', '12th Avenue', 2 )
	
	INSERT INTO Address ( number, postalCode, street, id )
	VALUES ( '18B', '4007B', '18th Avenue', 3 )
	
	INSERT INTO Person ( registrationNumber, id )
	VALUES ( 'DEF-456', 4 )
	
	INSERT INTO Person_Address ( owners_id, addresses_id )
	VALUES ( 1, 2 )
	
	INSERT INTO Person_Address ( owners_id, addresses_id )
	VALUES ( 1, 3 )
	
	INSERT INTO Person_Address ( owners_id, addresses_id )
	VALUES ( 4, 2 )
	
	DELETE FROM Person_Address
	WHERE  owners_id = 1
	
	INSERT INTO Person_Address ( owners_id, addresses_id )
	VALUES ( 1, 3 )
	
	
#### 4.3 双向 many-to-many with a link entity

- 实体：


	@Entity(name = "Person")
	public static class Person implements Serializable {
	
	    @Id
	    @GeneratedValue
	    private Long id;
	
	    @NaturalId
	    private String registrationNumber;
	
	    @OneToMany(mappedBy = "person", cascade = CascadeType.ALL, orphanRemoval = true)
	    private List<PersonAddress> addresses = new ArrayList<>();
	
	    public Person() {
	    }
	
	    public Person(String registrationNumber) {
	        this.registrationNumber = registrationNumber;
	    }
	
	    public Long getId() {
	        return id;
	    }
	
	    public List<PersonAddress> getAddresses() {
	        return addresses;
	    }
	
	    public void addAddress(Address address) {
	        PersonAddress personAddress = new PersonAddress( this, address );
	        addresses.add( personAddress );
	        address.getOwners().add( personAddress );
	    }
	
	    public void removeAddress(Address address) {
	        PersonAddress personAddress = new PersonAddress( this, address );
	        address.getOwners().remove( personAddress );
	        addresses.remove( personAddress );
	        personAddress.setPerson( null );
	        personAddress.setAddress( null );
	    }
	
	    @Override
	    public boolean equals(Object o) {
	        if ( this == o ) {
	            return true;
	        }
	        if ( o == null || getClass() != o.getClass() ) {
	            return false;
	        }
	        Person person = (Person) o;
	        return Objects.equals( registrationNumber, person.registrationNumber );
	    }
	
	    @Override
	    public int hashCode() {
	        return Objects.hash( registrationNumber );
	    }
	}
	
---	
	@Entity(name = "PersonAddress")
	public static class PersonAddress implements Serializable {
	
	    @Id
	    @ManyToOne
	    private Person person;
	
	    @Id
	    @ManyToOne
	    private Address address;
	
	    public PersonAddress() {
	    }
	
	    public PersonAddress(Person person, Address address) {
	        this.person = person;
	        this.address = address;
	    }
	
	    public Person getPerson() {
	        return person;
	    }
	
	    public void setPerson(Person person) {
	        this.person = person;
	    }
	
	    public Address getAddress() {
	        return address;
	    }
	
	    public void setAddress(Address address) {
	        this.address = address;
	    }
	
	    @Override
	    public boolean equals(Object o) {
	        if ( this == o ) {
	            return true;
	        }
	        if ( o == null || getClass() != o.getClass() ) {
	            return false;
	        }
	        PersonAddress that = (PersonAddress) o;
	        return Objects.equals( person, that.person ) &&
	                Objects.equals( address, that.address );
	    }
	
	    @Override
	    public int hashCode() {
	        return Objects.hash( person, address );
	    }
	}
	
---	
	@Entity(name = "Address")
	public static class Address implements Serializable {
	
	    @Id
	    @GeneratedValue
	    private Long id;
	
	    private String street;
	
	    @Column(name = "`number`")
	    private String number;
	
	    private String postalCode;
	
	    @OneToMany(mappedBy = "address", cascade = CascadeType.ALL, orphanRemoval = true)
	    private List<PersonAddress> owners = new ArrayList<>();
	
	    public Address() {
	    }
	
	    public Address(String street, String number, String postalCode) {
	        this.street = street;
	        this.number = number;
	        this.postalCode = postalCode;
	    }
	
	    public Long getId() {
	        return id;
	    }
	
	    public String getStreet() {
	        return street;
	    }
	
	    public String getNumber() {
	        return number;
	    }
	
	    public String getPostalCode() {
	        return postalCode;
	    }
	
	    public List<PersonAddress> getOwners() {
	        return owners;
	    }
	
	    @Override
	    public boolean equals(Object o) {
	        if ( this == o ) {
	            return true;
	        }
	        if ( o == null || getClass() != o.getClass() ) {
	            return false;
	        }
	        Address address = (Address) o;
	        return Objects.equals( street, address.street ) &&
	                Objects.equals( number, address.number ) &&
	                Objects.equals( postalCode, address.postalCode );
	    }
	
	    @Override
	    public int hashCode() {
	        return Objects.hash( street, number, postalCode );
	    }
	}
	
- *生成sql*：


	CREATE TABLE Address (
	    id BIGINT NOT NULL ,
	    number VARCHAR(255) ,
	    postalCode VARCHAR(255) ,
	    street VARCHAR(255) ,
	    PRIMARY KEY ( id )
	)
	
	CREATE TABLE Person (
	    id BIGINT NOT NULL ,
	    registrationNumber VARCHAR(255) ,
	    PRIMARY KEY ( id )
	)
	
	CREATE TABLE PersonAddress (
	    person_id BIGINT NOT NULL ,
	    address_id BIGINT NOT NULL ,
	    PRIMARY KEY ( person_id, address_id )
	)
	
	ALTER TABLE Person
	ADD CONSTRAINT UK_23enodonj49jm8uwec4i7y37f
	UNIQUE (registrationNumber)
	
	ALTER TABLE PersonAddress
	ADD CONSTRAINT FK8b3lru5fyej1aarjflamwghqq
	FOREIGN KEY (person_id) REFERENCES Person
	
	ALTER TABLE PersonAddress
	ADD CONSTRAINT FK7p69mgialumhegyl4byrh65jk
	FOREIGN KEY (address_id) REFERENCES Address
	
- *操作*：


	Person person1 = new Person( "ABC-123" );
	Person person2 = new Person( "DEF-456" );
	
	Address address1 = new Address( "12th Avenue", "12A", "4005A" );
	Address address2 = new Address( "18th Avenue", "18B", "4007B" );
	
	entityManager.persist( person1 );
	entityManager.persist( person2 );
	
	entityManager.persist( address1 );
	entityManager.persist( address2 );
	
	person1.addAddress( address1 );
	person1.addAddress( address2 );
	
	person2.addAddress( address1 );
	
	entityManager.flush();
	
	log.info( "Removing address" );
	person1.removeAddress( address1 );
	---------------------------------------------------
	INSERT  INTO Person ( registrationNumber, id )
	VALUES  ( 'ABC-123', 1 )
	
	INSERT  INTO Person ( registrationNumber, id )
	VALUES  ( 'DEF-456', 2 )
	
	INSERT  INTO Address ( number, postalCode, street, id )
	VALUES  ( '12A', '4005A', '12th Avenue', 3 )
	
	INSERT  INTO Address ( number, postalCode, street, id )
	VALUES  ( '18B', '4007B', '18th Avenue', 4 )
	
	INSERT  INTO PersonAddress ( person_id, address_id )
	VALUES  ( 1, 3 )
	
	INSERT  INTO PersonAddress ( person_id, address_id )
	VALUES  ( 1, 4 )
	
	INSERT  INTO PersonAddress ( person_id, address_id )
	VALUES  ( 2, 3 )
	
	DELETE  FROM PersonAddress
	WHERE   person_id = 1 AND address_id = 3
	

	