# About
A better Java wrapper around the
[Kyoto Cabinet](http://fallabs.com/kyotocabinet/ "Kyoto Cabinet: a straightforwardimplementation of DBM")
library. It's great to be able to easily access kyoto-cabinet from Java, however the
[default Java bindings](http://fallabs.com/kyotocabinet/javadoc/ "kyotocabinet-java Javadoc")
are missing some features we've come to expect in a modern Java developement environment. lastcommons-kyoto addresses
this by wrapping the default bindings in an API that should be more familiar to most Java developers.

# Dependencies
This wrapper uses `kyotocabinet-java` version
[1.24](http://fallabs.com/kyotocabinet/javapkg/kyotocabinet-java-1.24.tar.gz "kyotocabinet-java packages")
which in turn requires `kyotocabinet` version [
1.2.65 or greater](http://fallabs.com/kyotocabinet/pkg/ "kyotocabinet packages").
At this time you may need to build both the native libraries yourself. The `kyotocabinet-java` JNI bindings are in Maven Central.

# Start using
You can [download](https://github.com/lastfm/lastcommons-kyoto/downloads) a JAR file or obtain lastcommons-kyoto from
Maven Central using the following identifier:

* [fm.last.commons:lastcommons-kyoto:1.24.0](http://search.maven.org/#artifactdetails%7Cfm.last.commons%7Clastcommons-kyoto%7C1.24.0%7Cjar)
                                            
# Features
* Cleaner, more Java-like API.
* Error conditions represented with exceptions instead of magic return values.
* Implicit file suffix handling for the different Kyoto database types.
* Descriptive builder pattern for creating and validating Kyoto database configurations.

# Usage
**Note:** We have taken the liberty of statically importing some enum constants for readability.
#### Create a new file hash database:
```java
File dbFile = FILE_HASH.createFile("my-new-db");
KyotoDb db = new KyotoDbBuilder(dbFile)
  .modes(READ_WRITE)
  .buckets(42000)
  .memoryMapSize(2, MEBIBYTES)
  .compressor(LZO)
  .build();
db.open();
```
#### Create a new cache tree database:
```java
KyotoDb db = new KyotoDbBuilder(CACHE_TREE).build();
```
#### Open an existing file tree database:
```java
KyotoDb db = new KyotoDbBuilder("another-db.kct")
  .modes(READ_ONLY)
  .memoryMapSizeFromFile()
  .build();
```
#### Resources implement `java.io.Closeable`
With Java 7:
```java
try (KyotoCursor cursor = db.cursor()) {
  ...
} catch (IOException e) {
  ...
}
```
or with Apache Commons IO:
```java
IOUtils.closeQuietly(db); // from Apache Commons IO
```
#### Work with exceptions - not error codes
```java
try {
  db.append(key, value); // returns void
} catch (KyotoException e) {
  // You decide what happens next!
}
```
#### Return values represent outcomes, not errors
```java
boolean recordAlreadyExists = db.putIfAbsent("myKey", "myValue");
long removed = db.remove(keys, ATOMIC);
long records = db.recordCount() // Never Long.MIN_VALUE, never < 0
```
#### Conversion from kyoto's 16 byte decimal representation
```java
db.set("doubleValue", 463.94738d);
db.increment("doubleValue", 0.00123d);
double value = db.getDouble("doubleValue"); // value == 463.94861000000003d
```
#### Clearer transaction management
```java
try {
  db.begin(Synchronization.PHYSICAL);
  // Do stuff
  db.commit();
} catch (KyotoException e) {
  db.rollback();
}
```
#### Validation of Kyoto database configuration
```java
File dbFile = FILE_HASH.newFile(parent, "an-existing-db");          
KyotoDb db = new KyotoDbBuilder(dbFile)
  .modes(READ_ONLY)
  .pageComparator(LEXICAL)
  .build();

// The call to pageComparator() will fail with an
// IllegalArgumentException as file-hash does not
// support the 'pcom' option.
```
#### Hadoop-like MapReduce wrapper
```java
// A classic word count across the values 
new MapReduceJob(new Mapper() {
  public void map(byte[] key, byte[] value, Context context) {
    String[] words = new String(value).split(" ");
    for (String word : words) {
      context.write(word.getBytes(), new byte[] { 1 });
    }
  }
}, new Reducer() {
  public void reduce(byte[] key, Iterable<byte[]> values) {
    int count = 0;
    for (byte[] value : values) {
      count += value[0];
    }
    // output key and count
  }
}).executeWith(db);
```
# Building
This project uses the [Maven](http://maven.apache.org/) build system. See notes in the 'Dependencies' section on building the dependencies.

# Further work
Implement a Spring [`PlatformTransactionManager`](http://static.springsource.org/spring/docs/3.1.x/javadoc-api/org/springframework/transaction/PlatformTransactionManager.html "Spring Framework Javadoc - PlatformTransactionManager") for simple integration with Spring's transaction management framework.

# Contributing
All contributions are welcome. Please use the [Last.fm codeformatting profile](https://github.com/lastfm/lastfm-oss-config/blob/master/src/main/resources/fm/last/last.fm.eclipse-codeformatter-profile.xml) found in the `lastfm-oss-config` project for formatting your changes.

# Legal
Copyright 2012 [Last.fm](http://www.last.fm/)

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
 
[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)
 
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.




#############################################


##


Kyoto Cabinet: a straightforward implementation of DBM
Copyright (C) 2009-2012 Mikio Hirabayashi
Last Update: Fri, 04 Mar 2011 23:07:26 -0800
Kyoto Cabinet

There's a successor: Tkrzw. I recommend you to try it.
Overview

Kyoto Cabinet is a library of routines for managing a database. The database is a simple data file containing records, each is a pair of a key and a value. Every key and value is serial bytes with variable length. Both binary data and character string can be used as a key and a value. Each key must be unique within a database. There is neither concept of data tables nor data types. Records are organized in hash table or B+ tree.

Kyoto Cabinet runs very fast. For example, elapsed time to store one million records is 0.9 seconds for hash database, and 1.1 seconds for B+ tree database. Moreover, the size of database is very small. For example, overhead for a record is 16 bytes for hash database, and 4 bytes for B+ tree database. Furthermore, scalability of Kyoto Cabinet is great. The database size can be up to 8EB (9.22e18 bytes).

Kyoto Cabinet is written in the C++ language, and provided as API of C++, C, Java, Python, Ruby, Perl, and Lua. Kyoto Cabinet is available on platforms which have API conforming to C++03 with the TR1 library extensions. Kyoto Cabinet is a free software licensed under the GNU General Public License.
Documents

The following are documents of Kyoto Cabinet. They are contained also in the source package.

    Fundamental Specifications
    Specifications of Command Line Utilities
    Presentation
    API Documents of the core library (C/C++)

    API Documents for Java
    API Documents for Python 3.x
    API Documents for Python 2.x
    API Documents for Ruby
    API Documents for Perl
    API Documents for Lua

Packages

The following are the source packages of Kyoto Cabinet. As for binary packages, see the site of each distributor.

    Source Packages of the core library (C/C++)

    Source Packages for Java
    Source Packages for Python 3.x
    Source Packages for Python 2.x
    Source Packages for Ruby
    Source Packages for Perl
    Source Packages for Lua

    Binary Packages for Windows (C/C++/Java)

Information

Kyoto Cabinet was written and is maintained by Mikio Hirabayashi. You can contact the author by e-mail to `mikio@gmail.com'.

The following is a sibling project of Kyoto Cabinet.

    Remote Service (Kyoto Tycoon)



