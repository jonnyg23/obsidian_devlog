---
tags: 📝/🌱
aliases:
  -
cssclass:
---

# [[AWS DynamoDB]]

---

- When Adding or Updating items from the db, the attributes (if a [Reserved Word](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ReservedWords.html) ) must be configured correctly. Configuration includes using `ExpressionAttributeNames` like the following:

```python
response = self.table.update_item(
	Key={"partition-key": str(item_id)},

	UpdateExpression="""
		set #d = :d,
			#n = :name,
	""",
	ExpressionAttributeNames={
		"#d": "data",
		"#n": "name",
	},
	ExpressionAttributeValues={
		":d": data,
		":name": data["name"],
	},

	ReturnValues="ALL_NEW",
)
return response["Attributes"]
```


---



Tags:

Reference:

Related:


🔗 Links to this page:

