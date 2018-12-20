# CouchDB2

Slim Python interface module to CouchDB v2.x.

Relies on `requests`: http://docs.python-requests.org/en/master/

## Installation

Quick-and-dirty method, since this module is not yet on PyPi:
```
pip install [-e] git+https://github.com/pekrau/CouchDB2.git#egg=couchdb2
```

## Server
```python
Server(self, href='http://localhost:5984/', username=None, password=None)
```
Connection to the CouchDB server.
### \_\_str\_\_
```python
Server.__str__(self)
```
Return a simple string representation of the server interface.

### \_\_len\_\_
```python
Server.__len__(self)
```
Return the number of user-defined databases.

### \_\_iter\_\_
```python
Server.__iter__(self)
```
Return an iterator over all user-defined databases on the server.

### \_\_getitem\_\_
```python
Server.__getitem__(self, name)
```
Get the named database.
- Raises NotFoundError if no such database.

### \_\_contains\_\_
```python
Server.__contains__(self, name)
```
Does the named database exist?

### get
```python
Server.get(self, name, check=True)
```
Get the named database.
- Raises NotFoundError if 'check' is True and no database exists.

### create
```python
Server.create(self, name)
```
Create the named database.
- Raises BadRequestError if the name is invalid.
- Raises AuthorizationError if not server admin privileges.
- Raises CreationError if a database with that name already exists.
- Raises IOError if there is some other error.

## Database
```python
Database(self, server, name, check=True)
```
Interface to a named CouchDB database.

### \_\_str\_\_
```python
Server.__str__(self)
```
Return the name of the CouchDB database.

### \_\_len\_\_
```python
Server.__len__(self)
```
Return the number of documents in the database.

### \_\_contains\_\_
```python
Server.__contains__(self, id)
```
Does a document with the given id exist in the database?
- Raises AuthorizationError if not privileged to read.
- Raises IOError if something else went wrong.

### \_\_iter\_\_
```python
Server.__iter__(self)
```
Iterate over all documents in the database.

### \_\_getitem\_\_
```python
Server.__getitem__(self, id)
```
Return the document with the given id.
- Raises AuthorizationError if not privileged to read.
- Raises NotFoundError if no such document or database.
- Raises IOError if something else went wrong.

### exists
```python
Database.exists(self)
```
Does this database exist?
### check
```python
Database.check(self)
```
- Raises NotFoundError if this database does not exist.
### create
```python
Database.create(self)
```
Create this database.
- Raises BadRequestError if the name is invalid.
- Raises AuthorizationError if not server admin privileges.
- Raises CreationError if a database with that name already exists.
- Raises IOError if there is some other error.

### destroy
```python
Database.destroy(self)
```
Delete this database and all its contents.
- Raises AuthorizationError if not server admin privileges.
- Raises NotFoundError if no such database.
- Raises IOError if there is some other error.

### compact
```python
Database.compact(self)
```
Compact the database on disk. May take some time.
### is_compact_running
```python
Database.is_compact_running(self)
```
Is a compact operation running?
### get
```python
Database.get(self, id, rev=None, revs_info=False, default=None)
```
Return the document with the given id.
- Returns the default if not found.
- Raises AuthorizationError if not read privilege.
- Raises IOError if there is some other error.

### save
```python
Database.save(self, doc)
```
Insert or update the document.

If the document does not contain an item '_id', it is added
having a UUID4 value. The '_rev' item is added or updated.

- Raises NotFoundError if the database does not exist.
- Raises AuthorizationError if not privileged to write.
- Raises RevisionError if the '_rev' item does not match.
- Raises IOError if something else went wrong.

### delete
```python
Database.delete(self, doc)
```
Delete the document.
- Raises NotFoundError if no such document or no '_id' item.
- Raises RevisionError if no '_rev' item, or it does not match.
- Raises ValueError if the request body or parameters are invalid.
- Raises IOError if something else went wrong.

### load_design
```python
Database.load_design(self, name, doc, rebuild=True)
```
Load the design document under the given name.

If the existing design document is identical, no action is taken and
False is returned, else the document is updated and True is returned.

If 'rebuild' is True, force view indexes to be rebuilt after update.

Example of doc:
```
  {'views':
    {'name':
      {'map': "function (doc) {emit(doc.name, null);}"},
     'name_sum':
      {'map': "function (doc) {emit(doc.name, 1);}",
       'reduce': '_sum'},
     'name_count':
      {'map': "function (doc) {emit(doc.name, null);}",
       'reduce': '_count'}
  }}
```

More info: http://docs.couchdb.org/en/latest/api/ddoc/common.html

- Raises AuthorizationError if not privileged to read.
- Raise NotFoundError if no such database.
- Raises IOError if something else went wrong.

### view
```python
Database.view(self, designname, viewname, key=None, keys=None, startkey=None, endkey=None, skip=None, limit=None, sorted=True, descending=False, group=False, group_level=None, reduce=None, include_docs=False)
```
Return the selected rows from the named design view.

A `ViewResult` object is returned, containing the following attributes:
- `rows`: the list of `Row` objects.
- `offset`: the offset used for this set of rows.
- `total_rows`: the total number of rows selected.

A `Row` object contains the following attributes:
- `id`: the identifier of the document, if any.
- `key`: the key for the index row.
- `value`: the value for the index row.
- `doc`: the document, if any.

### load_index
```python
Database.load_index(self, fields, id=None, name=None, selector=None)
```
Load a Mango index specification.

- 'fields' is a list of fields to index.
- 'id' is the design document name.
- 'name' is the view name.
- 'selector' is a partial filter selector.

Returns a dictionary with items 'id' (design document name),
'name' (index name) and 'result' ('created' or 'exists').

- Raises BadRequestError if the index is malformed.
- Raises AuthorizationError if not server admin privileges.
- Raises ServerError if there is an internal server error.

### find
```python
Database.find(self, selector, limit=None, skip=None, sort=None, fields=None, use_index=None, bookmark=None, update=None)
```
Select documents according to the Mango index selector.

Returns a dictionary with items 'docs', 'warning', 'execution_stats'
and 'bookmark'.

- Raises BadRequestError if the selector is malformed.
- Raises AuthorizationError if not privileged to read.
- Raises ServerError if there is an internal server error.

### put_attachment
```python
Database.put_attachment(self, doc, content, filename=None, content_type=None)
```
'content' is a string or a file-like object.

If no filename, then an attempt is made to get it from content object.

- Raises ValueError if no filename is available.

### get_attachment
```python
Database.get_attachment(self, doc, filename)
```
Return a file-like object containing the content of the attachment.
### dump
```python
Database.dump(self, filepath)
```
Dump the entire database to the named tar file.
If the filepath ends with '.gz', the tar file is gzip compressed.

The `_rev` item of each document is kept.

A tuple `(ndocs, nfiles)` is returned.

### undump
```python
Database.undump(self, filepath)
```
Load the named tar file, which must have been produced by `dump`.

NOTE: The documents are just added to the database, ignoring any
`_rev` items.

A tuple `(ndocs, nfiles)` is returned.

## CouchDB2Exception
```python
CouchDB2Exception(self)
```
Base CouchDB2 exception.
## NotFoundError
```python
NotFoundError(self)
```
No such entity exists.
## BadRequestError
```python
BadRequestError(self)
```
Invalid request; bad name, body or headers.
## CreationError
```python
CreationError(self)
```
Could not create the entity; it exists already.
## RevisionError
```python
RevisionError(self)
```
Wrong or missing '_rev' item in the document to save.
## AuthorizationError
```python
AuthorizationError(self)
```
Current user not authorized to perform the operation.
## ContentTypeError
```python
ContentTypeError(self)
```
Bad 'Content-Type' value in the request.
## ServerError
```python
ServerError(self)
```
Internal server error.
## Row
```python
Row(self)
```
Row(id, key, value, doc)
### doc
Alias for field number 3
### id
Alias for field number 0
### key
Alias for field number 1
### value
Alias for field number 2
## ViewResult
```python
ViewResult(self)
```
ViewResult(rows, offset, total_rows)
### offset
Alias for field number 1
### rows
Alias for field number 0
### total_rows
Alias for field number 2
