# zstorage

A ZK-based object storage library

## Overview

This library facilitates the storage of data in Zookeeper.

At a high level, the Store supports the following API:

	Put(item Item) (Ident, error)
	Get(ident Ident) (item Item, err error)
	List(category string) (locations []Location, err error)
	Variants(location Location) (variants []string, err error)
	Delete(ident Ident) error
	Close() error

## Nomenclature

An Item fully represents an item in the datastore.  Clients will provide an Item when Putting data and will receive an Item when Getting data.

An Item is fully identified by a composed Ident.  The Ident points to a Location and also an optional Variant and an optional Version.

## Variants

An item may have any number of variants. Note that this is different from the Version which is described below.  If a client Puts an item with a variant, it will live as a child of the current node for that item.

Additionally, when deleting an item, the user may specify a variant.  If no variant is specified when deleting an item, that item and all of its variant will be deleted.

## Paths

Here is an example of how a typical path might look like in the system:

	/[basePath]/[category]/buckets/[bucket]/[name]/[variant]
	
The `basePath` can be configured on the Store, and will be prepended to any znode path that is generated for an operation.

The `category`, `name`, and `variant` parts are set on the item's Ident property.

The `bucket` is generated by the Store using a configurable hashing function.  It is derived by hashing the item `name`.  The number of buckets to which a name might possibly hash can be configured when constructing the store.

## Optimistic Locking

Part of an item's Ident is a `Version` field called `Version`.

When performing a Get, the Store will set the `Version` on the returned Item.  This allows the client to specify that version when performing a subsequent mutating operation.

If this is set on an item's Ident property when performing a mutating operation, it will ensure that the item that is updated is specifically that version when setting it.  If another client happened to set the item before the first client was able to do so, the library will return the `ErrVersionConflict` error.  At this point, the client may choose to do another read and try again.

If the `Item.Ident.Version` is set to `NoPriorVersion` when passing an Item to Put() it is assumed that this Put() must create the item and it will return ErrVersionConflict if the node already exists. If no Version is specified, Put will create the node if it doesn't already exist or ignore and overwrite the existing Item with the new one if it does.
