# Log format

<dl>
	<dt>Namespace</dt><dd>`log`</dd>
	<dt>Version</dt><dd>1.0</dd>
</dl>

The log format is for storing chronologically sorted data entries. Some common
uses for this data structure include chatrooms, diaries, or microblogs. This
format is designed to be generic so as to accommodate many different use-cases,
and provides a recommended way to extend the format with additional data.

## Overview

This format allows a single share to hold many named logs, each of which holds
many chronologically ordered entries.

Each entry belongs to a single author. The entry may or may not have an
attachment. An entry may or may not be ephemeral and be deleted after a certain
time.

The chosen name of this format ('log') has nothing to do with append-only logs.
In this log, items may be edited, overwritten, or new entries added which come
before existing entries.

## Logs

A share may have many logs, e.g:

```
+personalstuff.bxxx
	My Journal
	Dream Diary
	Tasty things I ate lately
```

Each of these logs is identified by a numerical ID:

```
+personalstuff.bxxx
	[0] My Journal
	[1] Dream Diary
	[2] Tasty things I ate lately
```

This log ID is used in the paths of the different documents which belong to a
log:

- A log entry document
- A log name document
- A log extension document

All of which have paths prefixed with the following:

`/log/1.0/{LOG_ID}/`

Where `{LOG_ID}` is the numerical ID of the log, e.g. `2`.

As log IDs only ever appear in the paths of log documents, a log's existence is
dependent on the existence of one of these log documents.

### Determining log IDs

When creating a document for a new log and determining the log ID for the path:

- Use `0` in case no documents have been created for any logs in the share.
- Determine the existing log ID with the highest numerical value, and increment
  it by 1.

The second option should be followed even if all the documents referring to the
highest log ID are empty.

## Log names

Each log may optionally have a human-readable name associated with it.

If a log does not have a document describing a human-readable name, clients may
choose to fall back to some other value (e.g. `Log 3`).

### Log name document path

The path form of a log name document is:

```
/log/1.0/{LOG_ID}/name
```

Where `{LOG_ID}` is replaced with the ID of the log that should have this name.

### Log name document fields

The `text` field of the document is the desired human-readable name of the log.
It can be any UTF-8 string.

If the value of the `text` field is an empty string, the document should be
treated the same way as if it were not there at all.

## Log entries

A log is made up of many entries, each belonging to a single author. Entries can
be overwritten or wiped, have an optional attachment, or may be ephemeral and
deleted after a certain time.

### Log entry path

The path format for an entry without an attachment is:

```
/log/1.0//{LOG_ID}/{TIMESTAMP}~{AUTHOR_ADDRESS}
```

Where `{LOG_ID}` is the numerical ID of the log this entry belongs to,
`{TIMESTAMP}` is the number of milliseconds elapsed since the UNIX epoch, and
`{AUTHOR_ADDRESS}` is the public address of the keypair which was used to sign
this document.

The path format for a non-ephemeral entry **with** an attachment is:

```
/log/1.0/{LOG_ID}/{TIMESTAMP}~{AUTHOR_ADDRESS}.{ATTACHMENT_EXTENSION}
```

This is the same as before, but is suffixed with `.{ATTACHMENT_EXTENSION}`,
where `{ATTACHMENT_EXTENSION}` is the appropriate file extension for the media
type of the attached data.

An ephemeral log entry document must place a `!` before the timestamp portion of
the path:

```
/log/1.0/{LOG_ID}/{TIMESTAMP}~{AUTHOR_ADDRESS}
/log/1.0/{LOG_ID}/!{TIMESTAMP}~{AUTHOR_ADDRESS}.{ATTACHMENT_EXTENSION}
```

The `TIMESTAMP` portion of the path serves two purposes: to help order the
documents chronologically, and to act as a 'created at' timestamp that persists
over subsequent edits.

The `AUTHOR_ADDRESS` portion is prefixed with a tilde, indicating to Earthstar
that the document at this path can only be written to by the author possessing
the keypair with that address.

### Log entry fields

- The `text` property of the document is the content of the entry. It can be any
  UTF-8 string.
  - If the value of the `text` field is an empty string, the document should be
    treated the same way as if it were not there at all.
- A log entry may optionally have an attachment, which can be a file of any
  type.
- If the entry is ephemeral, the `deleteAfter` field must be desired expiry time
  in UNIX microseconds.

## Extension documents

It is often desirable to associate extra attributes with a log and its entries.

For example, a log could be used to save memorable quotes of a child, where each
entry was a different quote. In our example we wish to extend the log format so
that a client application could display the current age of the child when each
quote was said. To do this, the client would need to know the child's birthday.

To create our extension, we would create a document at the following path:

```
log/1.0/37/ext/kid-quote/birthday
```

And the document would have the value of the `text` field set to something like
`2020-01-01`. Our client now has the birthday it needs to determine the age of
the child when the entry was created.

### Log extension path

The path format for a log extension document is:

```
log/1.0/{LOG_ID}/ext/{EXTENSION_NAMESPACE}/**
```

Where `LOG_ID` is the ID of the log being extended, and `EXTENSION_NAMESPACE` is
a string representing the extension form (e.g. `kid-quote`). The glob portion
conveys that after this, the path format is left to the discretion of the
extension in question, and may be as simple or complex as desired, with a single
subsequent path segment or many.

### Log extension fields

The fields of a log extension document are not proscribed and their requirements
are left to the designer of the log extension.