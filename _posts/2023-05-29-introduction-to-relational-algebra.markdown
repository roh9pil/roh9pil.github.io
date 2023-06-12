---
layout: post
title:  "Introduction to relational algebra"
date:   2023-05-29 12:25:00 +0900
categories: database sql realtiona_algebra
---

# Relational Algebra 는 무엇인가?

Relational Algebra 는 
* 사용자가 relational database 시스템과 소통하는데 사용하는 relational query language
* SQL (Structured Query Language) 의 이론적 기반
* Relational query language 는 functional query language

Relational query language 는 3가지 종류로 구분될 수 있습니다. 

1. Imperative query language
2. Functional query language
3. Declarative query language: tuple relational calculus

# Relation, attribute, 그리고 tuple

* Relation 은 테이블, attribute 는 칼럼, tuple 은 row

# Attribute 의 Domain 은?

* Function 배울 때, 그 domain (한국어로, 정의역)
* attribute 가 가질 수 있는 값의 집합
  - atomic: indivisible
  - nonatomic: object-relation database

# Relation 은?
* a subset of a cartesian product of a list of domains


# Key
* superkey: row 를 특정할 수 있는 attribute 의 집합
* candidate key: minimal superkey
* primary key: DB Designer 가 candidate key 중에서 선택하 것

# Fundamental Operations of Relational Algebra

* unary
  - Select: &sigma;
    - &sigma;<sub>predicate</sub>(relation)
  - Project: &Pi;
    - &Pi;<sub>attributes</sub>(relation)
  - Rename: &rho;
    - &rho;<sub>x</sub>(relation)
    - &rho;<sub>x(A<sub>1</sub>,A<sub>2</sub>,...,A<sub>n</sub>)</sub>(relation)
* binary
  - Union: &cup;
    - r &cup; s
    - r, s 의 attribute 갯수가 같고, 순서가 같은 attribute 이면 domain 도 동일해야 한다.
  - Set difference: - 
    - r - s
  - Cartesian product: &times;
    - r &times; s
 
# Relation algebra expression
* database 에 있는 relation
* fundamental operation 의 결과

# Additional Operations

* Join: &#10781;
  - r &#10781;<sub>&theta;</sub> s = &sigma;<sub>&theta;</sub>(r &times; s)
* Assignment: &#8592;
  - variable_name &leftarrow; relational_algebra_expression
* Division: &#247;
  - r &#247; s = &Pi;<sub>R-S</sub>(r) - &Pi;( (&Pi;<sub>R-S</sub>(r) &times; s) - &Pi;<sub>R-S,S</sub>(r))
  - Find all customers who have an account at all the branches


