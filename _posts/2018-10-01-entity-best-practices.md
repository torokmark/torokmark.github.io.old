---
layout: post
title:  "Best practices to write an Entity in Spring/Hibernate"
date:   2018-10-01 
categories: jekyll update
---

* Every entity has to have a *public default contstructor*
* `fetchType = FetchType.LAZY` on `@OneToOne`, `@OneToMany` and `@ManyToOne` and other associations (EAGER is the default except `@OneToMany`)
* If an association is not mandatory:
    - `optional = false` on association annotations
    - `@Column` and `@JoinColumn` should contain `nullable = false`
* Entity should implement `Persistable<IDTYPE>` interface 
    - Create own interface with default implementation:
         ```java
         public interface Identifiable extends Persistable<Long> { 
             default isNew() { 
                 return getId() == null; 
             }
         }`
         ```
* All ids should be boxed like `Long`, `Integer`, `String`, etc. (Implementing `Persistable<IDTYPE>` will ensure this.)
* Do not use *lombok* for entities (hibernate), generated code interfere with each other
* Do not reimplement `equals` and `hashCode` for entities
* Do not use primitives if they can be null in the DB

In the service layer:

* Use `@Transactional(readOnly = true)` if the method can write db or `@Transactional(readOnly = false)` if not (false is the default in spring)
