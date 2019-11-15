---
layout: post
title: Sqlalchemy complex queries and subqueries
description: 
date: '2019-11-15'
tags: programming
---

Here's how I put together a complex query in sqlalchemy using subqueries

## Approach

My brain already understands sql syntax so I choose something that reads like sql, but sqlalchemy has more than one syntax. This is the general syntax I use:

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

```

## Example 

The following excercise showcases: 

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

**Problem** 

Find the vendors and the domains with highest revenue that contributed to 90% of the total revenue in the past 7 days

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

    # second subquery will return only those domains that are in the top 90% (by keeping partial total)
    revenue_a = aliased(Revenue)
    revenue_b = aliased(Revenue)
    sub_query2 = session.query(
        revenue_a.domain_id, revenue_a.total_revenue
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
    ).order_by(
        revenue_a.total_revenue.desc()
    ).subquery()

    # everything together
    query = session.query(
        Vendor.id,
        Vendor.name,
        Domain.fqdn,
        sub_query2.c.total_revenue.label('revenue')
    ).select_from(
        Vendor
    ).join(
        Domain, Vendor.id == Domain.vendor_id
    ).join(
        sub_query2, sub_query2.c.domain_id == Domain.id
    ).order_by(
        sub_query2.c.total_revenue
    )
```

Generated sql: 

``` sql
SELECT
    vendor.id AS vendor_id,
    vendor.name AS vendor_name,
    domain.fqdn AS domain_fqdn,
    anon_1.total_revenue AS anon_1_total_revenue
FROM
    vendor
    JOIN domain ON vendor.id = domain.vendor_id
    JOIN (
        SELECT
            revenue_1.domain_id AS domain_id,
            revenue_1.total_revenue AS total_revenue
        FROM
            revenue AS revenue_1
            JOIN revenue AS revenue_2 ON revenue_2.total_revenue >= revenue_1.total_revenue
        WHERE
            revenue_1.total_revenue > ?
            AND revenue_2.total_revenue > ?
            AND datediff(CURRENT_TIMESTAMP, revenue_1.date) <= ?
        GROUP BY
            revenue_1.domain_id
        HAVING
            sum(revenue_2.total_revenue) <= (
                SELECT ? * SUM(revenue.total_revenue) AS anon_2 
                FROM revenue
                WHERE datediff(CURRENT_TIMESTAMP, revenue_1.date) <= ?
            )
        ORDER BY revenue_1.total_revenue DESC
    ) AS anon_1 ON anon_1.domain_id = domain.id
ORDER BY anon_1.total_revenue

```

This problem can be solved in more than one way and this not necessary the best way, I just dumped this example for future reference, the documention for sql alchemy is hard to navigate.


## Resources

Sqlalchemy docs

- query api [here](https://docs.sqlalchemy.org/en/13/orm/query.html)