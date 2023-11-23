How to implement index scan in cso-demo?

# User view

```
create table test(c1 int, c2 int);
insert into test values (101);...
insert into test values (102);
insert into test values (103);
... 10000;

create index index_1 on test(c1); (btree)
create hash index index_2 on test(c2);
select c1 + c3 from test where c1 = 5000 and c2 = 101 order by c2;
```

# index types
https://blog.csdn.net/qq_42768234/article/details/131458308

# implementation
## implementation of gporca
### information flow

metadata -> LogicalSelect(Exploration) -> LogicalIndexGet(Exploration) -> PhysicalIndexGet(implementation)  

why orca need to add logical index get? logicalGet -> physicaltablescan / physicalindexscan

### metadata define
```
IMDIndex
// object type
	virtual Emdtype
	MDType() const
	{
		return EmdtInd;
	}

	// is the index clustered
	virtual BOOL IsClustered() const = 0;

	// index type
	virtual EmdindexType IndexType() const = 0;

	// number of keys
	virtual ULONG Keys() const = 0;

	// return the n-th key column
	virtual ULONG KeyAt(ULONG pos) const = 0;

	// return the position of the key column
	virtual ULONG GetKeyPos(ULONG pos) const = 0;

	// number of included columns
	virtual ULONG IncludedCols() const = 0;

	// return the n-th included column
	virtual ULONG IncludedColAt(ULONG pos) const = 0;

	// return the position of the included column
	virtual ULONG GetIncludedColPos(ULONG column) const = 0;

	// part constraint
	virtual IMDPartConstraint *MDPartConstraint() const = 0;

	// type id of items returned by the index
	virtual IMDId *GetIndexRetItemTypeMdid() const = 0;

	// check if given scalar comparison can be used with the index key
	// at the specified position
	virtual BOOL IsCompatible(const IMDScalarOp *md_scalar_op,
							  ULONG key_pos) const = 0;

	// index type as a string value
	static const CWStringConst *GetDXLStr(EmdindexType index_type);

```
### work flow
```
CXformSelect2IndexGet::Transform
```

### scalar properties derive(required scalar properties)
```
// derive the scalar and relational properties to build set of required columns
	CColRefSet *pcrsOutput = pexpr->DeriveOutputColumns();
	CColRefSet *pcrsScalarExpr = pexprScalar->DeriveUsedColumns();
```
### index matching
```
CXformUtils::FIndexApplicable core-> key matching
```

exploration include tablescan, implementation()

optimization(enforcer + cost) choose indexscan 

## What should we do?

## target 1 
when add a new type of index, we should not change previous behavior of method. trait (method needed)

## target 2 
do we need index in exploration? because of statistic information derive? 
implementation -> logicalget2physicalindexscan -> statistic derivation.
logicalIndexGet

## target 3
do we need logicalSelect?     logicalScalarOperator(logicalGetOperator)  pattern matching

## target 4
logicalProperties(column, expression) scalarOperator ExprId

## todoList

metadata -> metadata of index trait (linbo)

exploration -> project(a + b scalarOperation list)
- 1、logicalProperties（LiBinfeng_01） scalarOperator(davidli2010) 
- 2、statistic derivation （linbo）
- 3、indexmatching（LiBinfeng_01） 
- 4、logicalIndexScan（LiBinfeng_01）interface/multiple indexscan in same table

implementation -> logical2physical（LiBinfeng_01）

optimization -> compute cost and choose index scan, physicalRequireProperties（LiBinfeng_01）



