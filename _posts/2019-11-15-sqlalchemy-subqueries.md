---
layout: post
title: Sqlalchemy complex queries and subqueries
description: 
date: '2019-11-15'
tags: programming
---

Here's how I put together a complex query in sqlalchemy using subqueries

## Approach

My brain already understands sql syntax so I choose something that reads like sql, however it's not the only syntax.

```
query or subquery = session.query(
  [select fields]
).select_from(
  [left_side]
).join(
  [right side or subquery]
).join(
  ....
).[other clauses]
[.subquery()]
```

## Example 

The following exercise showcases: 

 - aggregation
 - filtering, sorting
 - table aliasing 
 - column aliasing
 - sub queries

The model: 

```python
class Vendor(Base):
    __tablename__ = 'vendor'
    id = Column(Integer, primary_key=True)
    name = Column(String(100))
    status = Column(Integer)

class Domain(Base):
    __tablename__ = 'domain'
    id = Column(Integer, primary_key=True)
    fqdn = Column(String(255))
    vendor_id = Column(Integer, ForeignKey(Vendor.id))

class Revenue(Base):
    __tablename__ = 'revenue'
    id = Column(Integer, primary_key=True)
    date = Column(Date())
    domain_id = Column(Integer, ForeignKey(Domain.id))
    total_revenue = Column(Integer)
```

One `vendor` can have one or more `domain`s and `revenue` tracks the daily revenues per domain

**Problem** 

Find the vendors and the domains with highest revenue that contributed to 90% of the revenue of the past 7 days

```python
session = DBSession()

# first subquery to calculate 90% of revenue of last 7 days
sub_query = session.query(
    0.9 * func.sum(Revenue.total_revenue)
).select_from(
    Revenue
).filter(
    func.datediff(func.now(), Revenue.date) <= 7
).subquery()

# second subquery will return only those domains that are in the top 90% revenue 
# (using join >= and sum to calculate partial totals)
revenue_a = aliased(Revenue)
revenue_b = aliased(Revenuel)
sub_query2 = session.query(
    revenue_a.domain_id
).select_from(
    revenue_a
).join(
    revenue_b,
    revenue_b.total_revenue >= revenue_a.total_revenue
).filter(
    and_(revenue_a.total_revenue > 0, revenue_b.total_revenue > 0)
).filter(
    func.datediff(func.now(), revenue_a.date) <= 7
).group_by(
    revenue_a.domain_id
).having(
    func.sum(revenue_b.total_revenue) <= sub_query
).distinct().subquery()

# everything together
query = session.query(
    Vendor.id,
    Vendor.name,
    Domain.fqdn,
    func.sum(Revenue.total_revenue).label('revenue')
).select_from(
    Vendor
).join(
    Domain, Vendor.id == Domain.vendor_id
).join(
    Revenue, Revenue.domain_id == Domain.id
).join(
    sub_query2, sub_query2.c.domain_id == Domain.id
).group_by(
    Vendor.id, Domain.id, Domain.fqdn
)
```

Generated sql: 

``` sql
SELECT
    vendor.id AS vendor_id,
    vendor.name AS vendor_name,
    domain.fqdn AS domain_fqdn,
    sum(revenue.total_revenue) AS revenue
FROM
    vendor
    JOIN domain ON vendor.id = domain.vendor_id
    JOIN revenue ON revenue.domain_id = domain.id
    JOIN (
        SELECT DISTINCT
            revenue_1.domain_id AS domain_id
        FROM
            revenue AS revenue_1
            JOIN revenue AS revenue_2 ON revenue_2.total_revenue >= revenue_1.total_revenue
        WHERE
            revenue_1.total_revenue > ?
            AND revenue_2.total_revenue > ?
            AND datediff(CURRENT_TIMESTAMP, revenue_1.date) <= ?
        GROUP BY revenue_1.domain_id
        HAVING
            sum(revenue_2.total_revenue) <= (
                SELECT
                    ? * sum(revenue.total_revenue) AS anon_2
                FROM revenue
                WHERE
                    datediff(CURRENT_TIMESTAMP, revenue_1.date) <= ?
            )
    ) AS anon_1 ON anon_1.domain_id = domain.id
    GROUP BY
        vendor.id, domain.id, domain.fqdn

```

This problem can be solved in more than one way and this not necessary the best way, but it's complex enough to serve as reference for the syntax.


## Resources

Sqlalchemy docs

- query api [here](https://docs.sqlalchemy.org/en/13/orm/query.html)
- orm tutorial [here](https://docs.sqlalchemy.org/en/13/orm/tutorial.html)