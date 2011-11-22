-*- markdown -*-

# Clj-Liquibase v0.3

Clj-Liquibase is a simple Clojure wrapper around the Liquibase library for
carrying out relational database change management and migrations.

Supported actions:

* update
* tag
* rollback
* generate SQL for actions
* generate DB doc
* database diff


## Usage

Maven/Leiningen dependency details are here: [http://clojars.org/org.bituf/clj-liquibase](http://clojars.org/org.bituf/clj-liquibase)

Examples for usage can be found in the tutorial below:


## Building/Installation

You will need Maven 2 to build from sources. Execute the following:

    $ mvn clean package  # packages up a JAR in "target" dir
    $ mvn install        # to install to your local Maven repo
    $ mvn clojure:gendoc # generate Clojure API documentation


## License

   Copyright (C) 2011 Shantanu Kumar (kumar[dot]shantanu[at]gmail[dot]com)

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this software except in compliance with the License.
   You may obtain a copy of the License at

   [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.


# Documentation

You can use Clj-Liquibase after including its namespace in your project:

    (use 'org.bituf.clj-liquibase)
    (use 'org.bituf.clj-liquibase.change)


## Quickstart


Create a new project and include the following dependencies:


    [org.bituf/clj-dbcp "0.5"]
    [org.bituf/oss-jdbc "0.5"]
    [org.bituf/clj-liquibase "0.3"]


To use Clj-Liquibase you need to include the required namespace in your application and define a
changelog.


    ;; filename: fooapp/src/fooapp/dbchange.clj
    (ns fooapp.dbchange
      (:require
        [org.bituf.clj-liquibase :as lb]
        [org.bituf.clj-liquibase.change :as ch]))
    
    (def ct-change1 (ch/create-table :sample-table1
                      [[:id     :int          :null false :pk true :autoinc true]
                       [:name   [:varchar 40] :null false]
                       [:gender [:char 1]     :null false]]))
    
    (def changeset-1 ["id=1" "author=shantanu" [ct-change1]])
    
    (lb/defchangelog changelog [changeset-1])


After defining the changelog, you need to apply the changes (see below).


    ;; filename: fooapp/src/fooapp/dbmigrate.clj
    (ns fooapp.dbmigrate
      (:require
        [fooapp.dbchange         :as dbch]
        [org.bituf.clj-dbcp      :as dbcp]
        [org.bituf.clj-dbspec    :as spec]
        [org.bituf.clj-liquibase :as lb]))
    
    ;; define datasource for supported database using Clj-DBCP
    (def ds (dbcp/mysql-datasource "localhost" "dbname" "user" "pass"))
    
    (defn do-lb-action "Wrap f using DBSpec middleware and execute it"
      [f]
      (let [g (spec/wrap-dbspec (spec/make-dbspec ds)
                (lb/wrap-lb-init f))]
                (g)))
    
    (defn do-update "Invoke this function to update the database"
      []
      (do-lb-action #(lb/update dbch/changelog)))


Once you are done with these, you can invoke fooapp.dbmigrate/do-update to
carry out the changes.


## Reference

Please check The Bitumen Framework Handbook [https://bitbucket.org/kumarshantanu/bituf-handbook/src](https://bitbucket.org/kumarshantanu/bituf-handbook/src)