### Find users who can AddMembers to a group with admin privileges:
    MATCH  p=(A:User)-[:AddMember]->(B:Group)-[:AdminTo]->(C:Computer) RETURN p 

### Find Computers with Unconstrained Delegation:
    MATCH (n:Computer {UnconstrainedDelegation: True}) RETURN n

### Find a path from a user to a computer with Unconstrained Delegation (From @wald0):
    MATCH p = shortestPath((u:User {name:'MY USER@DOMAIN.COM'})-[r*1..]->(c:Computer {UnconstrainedDelegation:true})) RETURN p

### Find users who are an admin on a computer (from @wald0):
    MATCH (u1:User)-[r:AdminTo]->(c:Computer)
    RETURN u1.name, COUNT(DISTINCT(c))
    UNION ALL
    MATCH (u1:User)-[r1:MemberOf*1..6]->(g:Group)-[r2:AdminTo]->(c:Computer)
    RETURN u1.name, COUNT(DISTINCT(c))
    
Different output style (from @sadprocessor):

    MATCH 
    (U:User)-[r:MemberOf|:AdminTo*1..]->(C:Computer)
    WITH
    U.name as n,
    COUNT(DISTINCT(C)) as c 
    RETURN 
    {Name: n, Count: c} as SingleColumn
    ORDER BY c DESC

Find all users who are not domain admins, then find the total number of computers each of those users have admin rights to. Return the name of the user and the number of computers that user has admin rights to, ordered by number of computers in descending order (from @wald0):

    MATCH (u1:User)-[r1:MemberOf*1..]->(g:Group {name:'DOMAIN ADMINS@DOMAIN.COM'})
    WITH COLLECT(u1.name) as daUsers
    MATCH (u2:User)
    WHERE NOT u2.name in daUsers
    MATCH p1 = (u2)-[r2:AdminTo]->(c1:Computer)
    MATCH p2 = (u2)-[r3:MemberOf*1..]->(g1:Group)-[r4:AdminTo]->(c2:Computer)
    RETURN u2.name,COUNT(DISTINCT(c1)) + COUNT(DISTINCT(c2)) as computerCount
    ORDER BY computerCount DESC
