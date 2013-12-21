# clj-uuid

A Clojure library for generation and utilization of UUIDs (Universally Unique
Identifiers) as described by RFC-4122.  The essential nature of the
service it provides is that of an enormous _namespace_ and a
deterministic mathematical model by means of which one navigates it.

The provided namespace represents an _inexhaustable_ resource and as
such can be used in a variety of ways not feasible using traditional
techniques rooted in the notions imposed by finite resources.  When I
say "inexhaustable" this of course is slight hyperbolie, but not by
much.  The upper bound on the representation implemented by this
library limits the number of unique identifiers to a mere...

*three hundred forty undecillion two hundred eighty-two decillion three*
*hundred sixty-six nonillion nine hundred twenty octillion nine hundred thirty-eight*
*septillion four hundred sixty-three sextillion four hundred sixty-three*
*quintillion three hundred seventy-four quadrillion six hundred seven trillion*
*four hundred thirty-one billion seven hundred sixty-eight million two hundred*
*eleven thousand four hundred and fifty-five.*

If you think you might be starting to run low, let me know when you get down
to your last few undecillion or so and I'll see what I can do to help out.


## Motivation

UUIDs represent an extremely powerful and versatile computation
technique that is often overlooked, and underutilized. In my opinion,
this, in part, is due to the generally poor quality, performance, and
capability of available libraries and, in part, due to a general
misunderstanding in the popular consiousness of their proper use and
benefit.

In the process of exploring Clojure, coming from a background in
Common-Lisp, I was a bit dismayed by the current state of affairs
as regarding the facilities provided by the standard Java
implementation of RFC-4122 identifiers, java.util.UUID, and, thus
motivated, fell quickly to the the task of extending and improving on
the UUID situation as an ideal means to explore working with Clojure's
much vaunted "protocol" and "type" abstractions.

To a large extent, the design of the API and algorithmic
implementation is inspired by the Common-Lisp library
[_UNICLY_](http://github.com/mon-key/unicly) which is a painstakingly
optimized, encyclopaedic implementation of RFC-4122 the author of
which, Stan Pearman, has devoted years to researching, refining, and
improving.  To my knowledge there is no more performant, capable, and
precise implementation of the RFC-4122 specification available
anywhere, in any language, on any platform.

That having been said, Common-Lisp, as a platform for implementing a
facility of this nature, differs significantly from that of the JVM
and has an extremely extensive collection of functionality, representations,
and optimization techniques as part of its built-in standard library
that are specifically geared for projects of this nature.  Java, and
by extension, Clojure, on the other hand, are extremely limited in
this regard incurring significant limitations which must be overcome.
For example, there is no notion of unsigned numeric representation,
which is an essential requrement for implementing the bitwise
arithmetic operations used in the underlying UUID algorthims. So, I've
taken a modest approach with clj-uuid, focused on provision
of the completeness and correctness of the user api first, with the
intent to work incrementally toward matters of performance and
optimization over time.


## License

Copyright © 2013 Dan Lentz

Distributed under the Eclipse Public License either version 1.0 

## Usage

a UUID is output, IOW, sticklers should be careful when using
CL printing function which depend on dynamic value of *PRINT-CASE*!

UNICLY> (defparameter *unique-random-namespace* 
	   (uuid-princ-to-string  (unicly:make-v4-uuid))) 
;=> *UNIQUE-RANDOM-NAMESPACE*

UNICLY> *UNIQUE-RANDOM-NAMESPACE*
;=> "77b84745-ab13-49c6-8fdc-9afaabc51c52"

To convert this string back to a UUID use MAKE-UUID-FROM-STRING:

UNICLY> (setf *unique-random-namespace* 
	      (make-uuid-from-string *unique-random-namespace*))
;=> 77b84745-ab13-49c6-8fdc-9afaabc51c52

UNICLY> *unique-random-namespace*
;=> 77b84745-ab13-49c6-8fdc-9afaabc51c52

To print a UUID with a URN quailifier use UUID-AS-URN-STRING:

UNICLY> (uuid-as-urn-string nil *unique-random-namespace*)
;=> "urn:uuid:77b84745-ab13-49c6-8fdc-9afaabc51c52"

v4 UUIDs are fine so long as you don't need to persist an objects
identity and simply need a throw away or single session UUID.
Indeed, one could serialize/deserialize v4 UUIDs from a string to object
representation with each session if desired.

However, as indicated above a v4 UUID is best used as a "seed-value" for
generating a namespace which is unique to your application.

For persistent UUID solutions it is recommended to use MAKE-V5-UUID by
providing a persisted UUID namespace for an object to reside in.

You can make your own fabulous namespace like this:

UNICLY> (defparameter *my-fabulous-namespace* 
	 (make-v5-uuid *unique-random-namespace* "com.bubba.namespace"))
;=> *MY-FABULOUS-NAMESPACE*

UNICLY> *MY-FABULOUS-NAMESPACE*
;=> e5c2a048-863f-5c7d-a894-607070d2d299

Create some objects in the namespace *my-fabulous-namespace*:

UNICLY> (make-v5-uuid *my-fabulous-namespace* (namestring (user-homedir-pathname)))
;=> c0f2a167-dae7-55c0-ad57-1d8bad0444d3

UNICLY> (make-v5-uuid *my-fabulous-namespace* (namestring *default-pathname-defaults*))
;=> a5ace91c-d657-5f5c-abef-81bbef52d27c

UNICLY> (setf *default-pathname-defaults* (user-homedir-pathname))
;=> #P"/home/you/"

You should now find that the UUID for the CL:NAMESTRING of *DEFAULT-PATHNAME-DEFAULTS*
is the same as that of the namestring of USER-HOMEDIR-PATHNAME:

UNICLY> (make-v5-uuid *my-fabulous-namespace* (namestring *default-pathname-defaults*))
;=> c0f2a167-dae7-55c0-ad57-1d8bad0444d3

Note that each object returned by MAKE-V5-UUID has unique identity under CL:EQUALP:

UNICLY> (equalp (make-v5-uuid *my-fabulous-namespace* (namestring (user-homedir-pathname)))
                (make-v5-uuid *my-fabulous-namespace* (namestring *default-pathname-defaults*)))
;=> NIL

To tests equality among two UUIDs (even where their CL:PRINT-OBJECT is
identical) one must first convert the UUID to an intermediary format and compare
the identity of the intermediate formats instead.

One way to do this is test CL:EQUAL for two UUIDs using their string representation:

UNICLY> (equal (uuid-princ-to-string 
		 (make-v5-uuid *my-fabulous-namespace* (namestring (user-homedir-pathname))))
		(uuid-princ-to-string 
		 (make-v5-uuid *my-fabulous-namespace* (namestring *default-pathname-defaults*))))
;=> T

CL:EQUAL finds the two UUIDs above as having identical string representations.
However, checking string values for object identity is ugly b/c internally UUID
objects are represented as unsigned integer values.

Unicly provides features for comparing UUID representations in various
intermediary formats other than as strings and further below we present some
examples of Unicly's representations of UUIDs in forms other than strings and
illustrate some cleaner ways to interrogate UUID equality.

So, now that you've got a handle on a fabulous UUID namespace how do you persist it?
The quick and dirty way is to write the UUID string representation of
*my-fabulous-namespace* to a file somewhere.

UNICLY> (with-open-file (persist (make-pathname :directory '(:absolute "tmp") 
                                                :name "persisted-fabulous-namespace"
                                                :type "uuid")
                                 :direction :output
                                 :if-exists :supersede
                                 :if-does-not-exist :create)
          ;; Here we CL:PRIN1 the UUID string representation.
	  ;; This is for illustrative purposes, there are other ways.
          (prin1 *MY-FABULOUS-NAMESPACE*  persist))
;=> "e5c2a048-863f-5c7d-a894-607070d2d299"

UNICLY> (setf *my-fabulous-namespace* nil)
;=> NIL

To restore the string representation of the persisted UUID into the
*my-fabulous-namespace* variable just read in the contents of the file:

UNICLY> (with-open-file (persist (make-pathname :directory
                                                '(:absolute "tmp")
                                                :name "persisted-fabulous-namespace"
                                                :type "uuid")
                                 :direction :input
                                 :if-does-not-exist :error)
          (setf *my-fabulous-namespace* 
                (make-uuid-from-string (read-line  persist))))
;=> e5c2a048-863f-5c7d-a894-607070d2d299

When serialzing/deserializing large numbers of UUIDs it may be more expedient to
use other intermediary representations of your UUIDs. Unicly provides interfaces
for reading, writing, and converting UUIDs across various representations
including bit-vectors, byte-arrays, 128-bit integers, strings, etc.

Following examples illustrate some more of the Unicly interface.

We use the value of the v4-uuid in the variable *unique-random-namespace*
defined earlier above, but feel free to substitute *my-fabulous-namespace* (or
equivalent).

Testing the equivalence of two UUID objects:

UNICLY> (uuid-eql 
         (make-v5-uuid *unique-random-namespace* "bubba")
         (make-v5-uuid *unique-random-namespace* "bubba"))
;=> T

Printing a UUID object in hex-string-36 format:

UNICLY> (uuid-princ-to-string (make-v5-uuid *unique-random-namespace* "bubba"))
;=> "065944a4-7566-53b2-811b-11a20e0bfed2"

Testing equivalence of two UUID objects where the first is generated using
MAKE-V5-UUID and the second is generated from an equivelent hex-string-36
representation:

UNICLY> (uuid-eql 
         (make-v5-uuid *unique-random-namespace* "bubba")
         (make-uuid-from-string "065944a4-7566-53b2-811b-11a20e0bfed2"))
;=> T

Binding a variable *another-unique-random-namespace* for use as a namespace.
We initally bind it to the hex-string-36 representation of a v4 UUID:

UNICLY> (defparameter *another-unique-random-namespace* 
          (uuid-princ-to-string (unicly:make-v4-uuid)))
;=> *ANOTHER-UNIQUE-RANDOM-NAMESPACE*

Binding the *another-unique-random-namespace* variable to a UUID object:
UNICLY> (setf *another-unique-random-namespace*
              (make-uuid-from-string *another-unique-random-namespace*))
;=> f65c8371-0c41-4913-96e6-8a917666aa51

Creating a container to hold 32 v5 UUIDs for 16 names each of which will occupy
two distinct namespaces:

UNICLY> (defparameter *v5-uuids-in-distinct-unique-random-namespaces* '()) 
;=> *V5-UUIDS-IN-DISTINCT-UNIQUE-RANDOM-NAMESPACES*

Adding 32 v5 UUIDS to the container where each is a cons with the head of each
cons a UUID object and the tail the name of some object in a namespace.
For each name we create two UUIDs one will occupy the namespace
*unique-random-namespace* the other will occupy the namepsace
*another-unique-random-namespace*:

UNICLY> (loop
           initially (setf *v5-uuids-in-distinct-unique-random-namespaces* nil)
           for bubba in (loop
                           for cnt from 0 below 16  
                           collect (format nil "bubba-~D" cnt))
           do (push (cons (make-v5-uuid *unique-random-namespace*  bubba) bubba)
                    *v5-uuids-in-distinct-unique-random-namespaces*)
           (push (cons (make-v5-uuid *another-unique-random-namespace* bubba) bubba)
                 *v5-uuids-in-distinct-unique-random-namespaces*)
           finally (return *v5-uuids-in-distinct-unique-random-namespaces* ))

;=> ((7c34b05e-d7a0-573e-baa2-7cc407532609 . "bubba-15")
;     (f7922a16-0b67-5329-87c9-71fdaa52c6c1 . "bubba-15")
;     { ... }
;     (7af9b747-e1f4-59b1-8f05-0acb70220817 . "bubba-0")
;     (f3228291-0a24-5a46-a9e2-7963d4671069 . "bubba-0"))

Retrieving the UUID for the name "bubba-8" in the namespace
*unique-random-namespace*:

UNICLY> (assoc 
         (make-v5-uuid *unique-random-namespace* "bubba-8")
         *v5-uuids-in-distinct-unique-random-namespaces*
         :test #'uuid-eql)
;=> (8e64c855-70fd-5d53-82ce-67e545f724a1 . "bubba-8")

Retrieving the UUID for the name "bubba-8" in the namespace
*another-unique-random-namespace*:

UNICLY> (assoc 
         (make-v5-uuid *another-unique-random-namespace* "bubba-8")
         *v5-uuids-in-distinct-unique-random-namespaces*
         :test #'uuid-eql)
;=> (ef74e326-4ecc-5edc-9b55-e69e6069610a . "bubba-8")

Testing if two identical names can be UUID-EQL when each occupies a different
namespace:
UNICLY> (uuid-eql 
         (make-v5-uuid *unique-random-namespace* "bubba-8")
         (make-v5-uuid *another-unique-random-namespace* "bubba-8"))
;=> NIL

Testing if two identical names can be UUID-EQL when each occupies the same
namespace:

UNICLY> (uuid-eql 
         (make-v5-uuid *unique-random-namespace* "bubba-8")
         (car (assoc 
               (make-v5-uuid *unique-random-namespace* "bubba-8")
               *v5-uuids-in-distinct-unique-random-namespaces*
               :test #'uuid-eql)))
;=> T

Examining the bit-vector representation of the *unique-random-namespace* UUID:

UNICLY> (uuid-to-bit-vector *unique-random-namespace*)
;=> #*01110111101110000100011101000101101010110001001101001001110001101000111111011100100110101111101010101011110001010001110001010010

Testing with UNIQUE-UNIVERSAL-IDENTIFIER-P whether the value of
*unique-random-namespace* is an instance of class UNIQUE-UNIVERSAL-IDENTIFIER:

UNICLY> (unique-universal-identifier-p *unique-random-namespace*)
;=> T

When testing an object with UNIQUE-UNIVERSAL-IDENTIFIER-P if the object is a
bit-vector and the form of that bit-vector satisfies UUID-BIT-VECTOR-128-P and
the appropriate version bit of the bit-vector is set, indication is given that
the bit-vector may be coerceable to an object which would satisfy
UNIQUE-UNIVERSAL-IDENTIFIER-P. This indication is provided as the CL:NTH-VALUE 1
as illustrated by the following return value:

UNICLY> (unique-universal-identifier-p (uuid-to-bit-vector *unique-random-namespace*))
;=> NIL, (UUID-BIT-VECTOR-128 4)

Testing whether the null-uuid satisfies UNIQUE-UNIVERSAL-IDENTIFIER-P:

UNICLY> (unique-universal-identifier-p (make-null-uuid))
;=> T

Converting a UUID to bit-vector representation with UUID-TO-BIT-VECTOR then
converting that to an integer value with UUID-BIT-VECTOR-TO-INTEGER:

UNICLY> (uuid-bit-vector-to-integer (uuid-to-bit-vector *unique-random-namespace*))
;=> 159134959691145724577639637335874542674

Converting a UUID to byte-array reresentation with UNICLY::UUID-TO-BYTE-ARRAY:

UNICLY> (unicly::uuid-to-byte-array *unique-random-namespace*)
;=> #(119 184 71 69 171 19 73 198 143 220 154 250 171 197 28 82)

Converting a UUID to byte-array reresentation with UNICLY::UUID-TO-BYTE-ARRAY
then converting that to a bit-vector:

UNICLY> (uuid-byte-array-to-bit-vector (unicly::uuid-to-byte-array *unique-random-namespace*))
;=> #*01110111101110000100011101000101101010110001001101001001110001101000111111011100100110101111101010101011110001010001110001010010

Note, above when converting the UUID object to a byte-array we used the internal
symbol UNICLY::UUID-TO-BYTE-ARRAY however the preferred interface for retrieving
the byte-array representation of a UUID object is UUID:GET-NAMESPACE-BYTES.  The
symbol UNICLY::UUID-TO-BYTE-ARRAY is not exported by Unicly b/c its
implementation conflicts with UUID:UUID-TO-BYTE-ARRAY (the two functions access
differently named slot values of their respective base classes
UNICLY:UNIQUE-UNIVERSAL-IDENTIFIER vs. UUID:UUID).

Testing if a UUID object is UUID-EQL to itself:

UNICLY> (uuid-eql *unique-random-namespace* *unique-random-namespace*)
;=> T

Testing if a UUID object is UUID-EQL to its bit-vector representation:

UNICLY> (uuid-eql *unique-random-namespace* (uuid-to-bit-vector *unique-random-namespace*))
;=> T

Testing if a UUID object is UUID-EQL to a copy of itself as per UUID-COPY-UUID:

UNICLY> (let ((copy (uuid-copy-uuid *unique-random-namespace*)))
          (uuid-eql copy *unique-random-namespace*))
;=> T

Testing if a UUID object is UUID-EQL to its byte-array representation.
Note, this is likely to change in future versions!

UNICLY> (uuid-eql (uuid-to-byte-array *unique-random-namespace*)
                  *unique-random-namespace*)
;=> NIL

Testing if two UUID bit-vector representations are UUID-BIT-VECTOR-EQL:

UNICLY> (uuid-bit-vector-eql 
         (uuid-to-bit-vector *unique-random-namespace*)
         (uuid-byte-array-to-bit-vector (unicly::uuid-to-byte-array *unique-random-namespace*)))
;=>T

Testing if two UUID bit-vector representations are UUID-EQL:

UNICLY> (uuid-eql 
         (uuid-to-bit-vector *unique-random-namespace*)
         (uuid-byte-array-to-bit-vector (unicly::uuid-to-byte-array *unique-random-namespace*)))
;=> T

Note, we can also test if two UUID bit-vector representations are CL:EQUAL.
We can not do the same for two UUID byte-array representations, instead we must
use CL:EQUALP:

UNICLY> (equal
         (uuid-to-bit-vector (make-v5-uuid *unique-random-namespace* "bubba"))
         (uuid-to-bit-vector (make-v5-uuid *unique-random-namespace* "bubba"))) 
;=> T

UNICLY> (equal
         (uuid-to-bit-vector (make-v5-uuid *unique-random-namespace* "bubba"))
         (uuid-to-bit-vector (make-v5-uuid *unique-random-namespace* "NOT-A-bubba")))
;=> NIL

UNICLY> (equal
         (uuid-get-namespace-bytes (make-v5-uuid *unique-random-namespace* "bubba"))
         (uuid-get-namespace-bytes (make-v5-uuid *unique-random-namespace* "bubba")))
;=> NIL

UNICLY> (equalp
         (uuid-get-namespace-bytes (make-v5-uuid *unique-random-namespace* "bubba"))
         (uuid-get-namespace-bytes (make-v5-uuid *unique-random-namespace* "bubba")))
;=> T

UNICLY> (equalp
         (uuid-get-namespace-bytes (make-v5-uuid *unique-random-namespace* "bubba"))
         (uuid-get-namespace-bytes (make-v5-uuid *unique-random-namespace* "NOT-A-BUBBA")))
;=> NIL

Roundtripping UUID representations:
 uuid -> bit-vector -> uuid -> byte-array -> bit-vector -> uuid 
  -> byte-array -> uuid -> uuid-string-36 -> uuid

First we verify the identity of the name "bubba" in the *uuid-namespace-dns*
namespace:

UNICLY> (make-v5-uuid *uuid-namespace-dns* "bubba")
;=> eea1105e-3681-5117-99b6-7b2b5fe1f3c7

Does the roundtripping return an equivalent object?:

UNICLY> (make-uuid-from-string
         (uuid-princ-to-string
          (uuid-from-byte-array
           (uuid-to-byte-array
            (uuid-from-bit-vector
             (uuid-byte-array-to-bit-vector
              (uuid-to-byte-array 
               (uuid-from-bit-vector 
                (uuid-to-bit-vector 
                 (make-v5-uuid *uuid-namespace-dns* "bubba"))))))))))
;=> eea1105e-3681-5117-99b6-7b2b5fe1f3c7

Comparing return value of UUID-EQL with CL builtin operators CL:EQ, CL:EQL,
CL:EQUAL, CL:EQUALP, and CL:SXHASH:

UNICLY> (let* ((uuid-1    (make-v5-uuid *uuid-namespace-dns* "bubba"))
               (uuid-1-bv (uuid-to-bit-vector (make-v5-uuid *uuid-namespace-dns* "bubba")))
               (uuid-2    (uuid-from-bit-vector uuid-1-bv)))
          (list :uuid-eql (uuid-eql uuid-1 uuid-2)
                :eq       (eq uuid-1 uuid-2)
                :eql      (eql uuid-1 uuid-2) 
                :equal    (equal uuid-1 uuid-2)
                :equalp   (equalp uuid-1 uuid-2)
                :sxhash   (list (sxhash uuid-1) (sxhash uuid-2))))
;=> (:UUID-EQL T :EQ NIL :EQL NIL :EQUAL NIL :EQUALP NIL :SXHASH (121011444 363948070))

Get the integer version of a UUID object:

UNICLY> (uuid-version-uuid *unique-random-namespace*)
;=> 4

UNICLY> (uuid-version-uuid (make-v5-uuid *unique-random-namespace* "bubba-8"))
;=> 5

Using a predicate to test the version of a UUID object:

UNICLY> (uuid-bit-vector-v4-p (uuid-to-bit-vector *unique-random-namespace*))
;=> T
 
UNICLY> (uuid-bit-vector-v5-p (uuid-to-bit-vector *unique-random-namespace*))
;=> NIL

Generating an instance of the null-uuid:

UNICLY> (make-null-uuid)
;=> 00000000-0000-0000-0000-000000000000

(Note, some special mojo occurs behind the curtains to ensure unique identity
for the null-uuid b/c the CL:SXHASH of the null-uuid is an intransient value).

MAKE-NULL-UUID is the preferred interface for accessing the null-uuid, we can
test if its return-value is an instance of class
UNIQUE-UNIVERSAL-IDENTIFIER-NULL with UNIQUE-UNIVERSAL-IDENTIFIER-NULL-P:

UNICLY> (unique-universal-identifier-null-p (make-null-uuid))
;=> T

Get the version of the null-uuid. Note, the CL:NTH-VALUE 1 can be checked to
verify that every bit of the UUID object is 0 (as opposed to an object with a
partial bit signature at bits 48-51 mimicing that of the null-uuid):

UNICLY> (uuid-version-uuid (make-null-uuid))
;=>  0, UNICLY::NULL-UUID

Testing if the null-uuid is UUID-EQL to itself:

UNICLY> (uuid-eql (make-null-uuid) (make-null-uuid))
;=> T

The UUID is sometimes referenced as having an 8:4:4:4:12 hex string representation. 
We refer to this representation as a UUID object with type UUID-HEX-STRING-36.

However, such references imply a string-centric view-point of the UUID when
really it is much saner to see the uuid as a sequence of bits or bytes.

Following table illustrates the components of a UUID as a bit/byte field. 
Note, it will not display correctly in a text-editor word/line wrapping is
enabled and/or your display is unable to lines of render text out to 140 columns
:{

The UUID as bit field:

 WEIGHT   INDEX      OCTETS                     BIT-FIELD-PER-OCTET
    4  | (0  31)  | 255 255 255 255         | #*11111111 #*11111111 #*11111111 #*11111111  | %uuid_time-low               | uuid-ub32
    2  | (32 47)  | 255 255                 | #*11111111 #*11111111                        | %uuid_time-mid               | uuid-ub16
    2  | (48 63)  | 255 255                 | #*11111111 #*11111111                        | %uuid_time-high-and-version  | uuid-ub16
    1  | (64 71)  | 255                     | #*11111111                                   | %uuid_clock-seq-and-reserved | uuid-ub8
    1  | (72 79)  | 255                     | #*11111111                                   | %uuid_clock-seq-low          | uuid-ub8
    6  | (80 127) | 255 255 255 255 255 255 | #*11111111 #*11111111 #*11111111 #*11111111 #*11111111 #*11111111 | %uuid_node | uuid-ub48

The UUIDs bit-vector representation:

UNICLY> (uuid-to-bit-vector (make-v5-uuid *uuid-namespace-dns* "bubba"))
;=> #*11101110101000010001000001011110001101101000000101010001000101111001100110110110011110110010101101011111111000011111001111000111
;     !      !       !       !       !       !       !       !        !      !       !       !       !       !       !       !       !  
;     0      7       15      23      31      39      47      55       63     71      79      87      95      103     111     119     127
;      --1--   --2--   --3--   --4--   --5--   --6--   --7--    --8--   --9--   -10-   -11-     -12-    -13-    -14-    -15-    -16-  
;     |  time-low slot               | time-mid slot | time-high slot | rsvd |  low  |                node slot                      |

The UUIDs binary integer representation:

UNICLY> #b11101110101000010001000001011110001101101000000101010001000101111001100110110110011110110010101101011111111000011111001111000111
;=> 317192554773903544674993329975922389959

The byte-array reresentation of a UUIDs integer representation:

UNICLY> (uuid-integer-128-to-byte-array 317192554773903544674993329975922389959)
;=> #(238 161 16 94 54 129 81 23 153 182 123 43 95 225 243 199)
 
UNICLY> (uuid-to-byte-array (make-v5-uuid *uuid-namespace-dns* "bubba"))
;=> #(238 161 16 94 54 129 81 23 153 182 123 43 95 225 243 199)

The component octet bit-vector reresentation of a UUID:

UNICLY> (map 'list #'uuid-octet-to-bit-vector-8
             (uuid-to-byte-array (make-v5-uuid *uuid-namespace-dns* "bubba")))
;=> (#*11101110 #*10100001 #*00010000 #*01011110 #*00110110 #*10000001 #*01010001 #*00010111 
;    #*10011001 #*10110110 #*01111011 #*00101011 #*01011111 #*11100001 #*11110011 #*11000111)

Converting from UUID -> byte-array -> bit-vector:

UNICLY> (uuid-byte-array-to-bit-vector (uuid-to-byte-array (make-v5-uuid *uuid-namespace-dns* "bubba")))
;=> #*11101110101000010001000001011110001101101000000101010001000101111001100110110110011110110010101101011111111000011111001111000111

The upper bounds of a UUID in binary integer representation:

UNICLY> #b11111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111
;=> 340282366920938463463374607431768211455
 
The number of unsigned bits used to represent the upper bounds of a UUIDs
integer representation:

UNICLY> (integer-length 340282366920938463463374607431768211455) 
;=> 128

The octet count of the upper bounds of a UUIDs integer representation:

UNICLY> (truncate (integer-length 340282366920938463463374607431768211455) 8)
;=> 16

The upper bounds of UUID in decimal integer representation (longform):

UNICLY> (format t "~R" 340282366920938463463374607431768211455)
;=> three hundred forty undecillion two hundred eighty-two decillion three hundred
;   sixty-six nonillion nine hundred twenty octillion nine hundred thirty-eight
;   septillion four hundred sixty-three sextillion four hundred sixty-three
;   quintillion three hundred seventy-four quadrillion six hundred seven trillion
;   four hundred thirty-one billion seven hundred sixty-eight million two hundred
;   eleven thousand four hundred fifty-five

Converting from a UUID bit-vector representation to an integer:

UNICLY> (uuid-bit-vector-to-integer (uuid-to-bit-vector (make-v5-uuid *uuid-namespace-dns* "bubba")))
;=> 317192554773903544674993329975922389959

Converting from a UUID byte-array representation to an integer:

UNICLY> (uuid-integer-128-to-byte-array 317192554773903544674993329975922389959)
;=> #(238 161 16 94 54 129 81 23 153 182 123 43 95 225 243 199)

Converting from a UUID byte-array representation to a  UUID integer representation:

UNICLY> (uuid-byte-array-16-to-integer 
         (uuid-integer-128-to-byte-array 317192554773903544674993329975922389959))
;=> 317192554773903544674993329975922389959

Converting from a UUID integer representation to a UUID bit-vector representation:

UNICLY> (uuid-integer-128-to-bit-vector 317192554773903544674993329975922389959)
;=> #*11101110101000010001000001011110001101101000000101010001000101111001100110110110011110110010101101011111111000011111001111000111

Testing if two UUIDs are UUID-BIT-VECTOR-EQL where the first is coerced to a
bit-vector from a UUID object and the second is coerced to a bit-vector from a
UUID integer representation:

UNICLY> (uuid-bit-vector-eql (uuid-to-bit-vector (make-v5-uuid *uuid-namespace-dns* "bubba")) 
                             (uuid-integer-128-to-bit-vector 317192554773903544674993329975922389959))
;=> T

Testing if two UUIDs are UUID-BIT-VECTOR-EQL where the first is coerced to a
bit-vector from UUID integer representation and the second is coerced to a
bit-vector from a UUID byte-array representation:

UNICLY> (uuid-bit-vector-eql (uuid-integer-128-to-bit-vector 317192554773903544674993329975922389959)
                             (uuid-byte-array-to-bit-vector (uuid-integer-128-to-byte-array 317192554773903544674993329975922389959)))
;=> T


    
Differences between the Unicly system and the uuid system:

Unicly has a similar interface to Boian Tzonev's Common Lisp library uuid: 
 :SEE (URL `https://github.com/dardoria/uuid')

Indeed, the core of Unicly is derived from Tzonev's uuid codebase.

However, Unicly deviates in some not insignificant ways from Tzonev's uuid and
while we have made some attempt to create a compatibility layer between the two
libraries the UUID objects generated with Unicly can not be used interchangeably
with those of Tzonev's uuid.

Some notable differences between Unicly and Tzonev's uuid:

 - Unicly is developed on SBCL 

   * Many routines are targeted towards making use of SBCL specific features.

   * It is highly declaration bound and inlined.

   * I do not test on implementations other than SBCL, but code for generating
     v3, v4, and v5 UUIDs *should* run portably on other Common Lisps ;}

 - Unicly is developed primarily for speedy minting of v3 and v5 UUIDs.
   On an x86-32 SBCL we have found Unicly's minting of v3 and v5 UUIDs to be
   significantly faster (at least 3-5x) than equivalent code from uuid.
   See unicly/unicly-timings.lisp for some timing comparisons.
   
   * Unicly is not particlulary faster than uuid when minting v4 UUIDS. 
   This is to be expected as both systems depend on frobbing *random-state*
   and there is little room for optimization beyond some internal declarations.
   
   * Unicly does however have different performace characteristcs when comparing
   timings of UNICLY:MAKE-V5-UUID with UUID:MAKE-V5-UUID. 

   Following timings were made using functionally identical namespaces for 1mil
   invocations on an x86-32 SBCL.

   Name components were taken from an array of 1mil elements where each element was
   a randomly generated string and where each string was between 1-36 characters long
   and where each character of the string was a randomly chosen UTF-8 characater
   (pulled from a constrained set of 360). With each invocation having the basic form:

    (unicly:make-v5-uuid <NAMESPACE> <RANDOM-NAME>)
    (uuid:make-v5-uuid <NAMESPACE> <RANDOM-NAME>)

    unicly:make-v5-uuid 
     18.251 seconds of real time
     54,614,814,653 processor cycles
     961,242,536 bytes consed
   
   uuid:make-v5-uuid
     57.404 seconds of real time
     171,781,583,768 processor cycles
     5,356,186,536 bytes consed
   
   The above ratios are similar for the equivalent MAKE-V3-UUID functions.
   
   Other significant performace differences can be seen between Unicly and uuid
   around the respective system's UUID-TO-BYTE-ARRAY, UUID-FROM-BYTE-ARRAY,
   MAKE-UUID-FROM-STRING functions.
   
   However, Unicly's biggest performance gains don't become apparent until using
   Unicly for equivalence tests and hash-table lookups using Unicly's extended
   interface as provided by UUID-EQL, UUID-BIT-VECTOR-EQL, etc.

 - Unicly is extended with support for creating/storing/(de)serializing UUID
   objects as bit vectors.

   * For persistence libraries which make use of hash-tables to store their keys
     as UUID objects in hex-string-36 representation there are some potentially big
     gains to be had by moving to a bit-vector base UUID representation.
   
     For example, on SBCL it is possible to SB-EXT:DEFINE-HASH-TABLE-TEST which
     tests for UUID bit-vector equivalence using UUID-BIT-VECTOR-EQL instead of
     CL:EQUAL and CL:EQUALP.
     (Underneath the covers UUID-BIT-VECTOR-EQL invokes SB-INT:BIT-VECTOR-=
      A nearly equivalent routine is provided for other CLs)

 - Unicly is extended with support for preserving identity of the null-uuid.
   This feature is specified in RFC 4122.

 - Unicly prints UUID string representations in case-significant form.
   This feature is specified in RFC 4122 Section 3. "Namespace Registration Template"
   as follows:

    ,----
    | The hexadecimal values "a" through "f" are output as lower case characters
    | and are case insensitive on input.
    `----

 - Unicly defines its base UUID class as UNIQUE-UNIVERSAL-IDENTIFIER instead of
   as the class UUID.

 - Unicly does not expose accessors for the slots of the UUID class
   UNIQUE-UNIVERSAL-IDENTIFIER.

 - Unicly slot-names for the base class UNIQUE-UNIVERSAL-IDENTIFIER are strongly
   namespaced with "%uuid_". This intent here is twofold:

   * Our opinion is that UUID identity should be considered immutable once minted.
     There should be no need for user code to directly modify a UUIDs slot
     values. Obfuscating easy access to the class slots of
     UNIQUE-UNIVERSAL-IDENTIFIER helps prevent this.

    * Because the Unicly interface is similar to that of the uuid library we've
      attempted to prevent trivial visual namespace collision with the slots of
      the uuid library.  Projects using both Unicly and the uuid library may
      benefit from being able to easily distinguish among the two.

 - Unicly's printing of a UUIDs string representation is not always conformant
   with ANSI spec.

    * The UUID CL:PRINT-OBJECT method for the class UNIQUE-UNIVERSAL-IDENTIFIER is
      not wrapped around PRINT-UNREADABLE-OBJECT.

 - Unicly's interface is extensively documented.

 - Unicly's source-code is commented with references to the relevant portions of
   RFC 4122.

 - Unicly does not have a dependency on trivial-utf-8

   * SBCL users can use internal features (assuming a Unicode enabled SBCL)

   * non-SBCL code can (and should) use flexi-streams instead 

 - Unicly is not released under an LLGPL licenses.

   * If licensing issues are a concern in your project please take a moment to
     investigate unicly/LICENSE.txt

    Although Unicly is initially derived from Tzonev's uuid library we note that
    significant portions of that library were in turn strongly derived from the
    non-normative reference implementation source code included of RFC4122 Appendix
    C as a wholly functional C language source code implementation of RFC4122.

    We believe the original RFC reference implementation and license have clear
    precedent in lieu of the later LLGPL and believe it reasonable to revert to the
    spirit of the original permissive and non-LLGPL'd license included of RFC4122.
   
 - Unicly is not targeted for generation of version 1 UUIDs (e.g. time based).

   The general implementation strategy for minting v1 UUID is reliant on
   interrogation of the system's underlying hardware and clock setting [1].
   When this is the strategy taken we have found that:

    * It requires platform and implementation specific code;

    * Minting version 1 UUIDs requires interrogating the MAC address of an
      Ethernet device;

    * Minting version 1 UUIDs requires interrogating the system clock -- there
      are in general some notoriously nasty bugs which spring from reliance on
      the value of the system clock e.g. cross-platform multi-boot systems...

    * Minting version 1 UUIDs is slow;

    * There is no portably universal mechanism for generation of version 1 UUIDs
      Some implementations use the hardware for seed value others use a random-number.

    * The uniqueness of version 1 UUIDs is not nearly as robust as the v3, v4,
      v5 variants. There are numerous mechanisms by which a v1 UUID can
      become corrupted which simply do not affect the others.

   [1]  RFC 4122 Section 4.5 "Node IDs that Do Not Identify the Host"
   Suggests that a v1 UUID may also be minted from a "47-bit cryptographic
   quality random number" by using it as the bottom 47 bits of the UUID Node id
   and setting the LSB of the first octet of the UUID node id to 1.
   Unfortunately, when attempting to implement this alternative strategy we found that
   Tzonev's uuid library has what we believe to be a bug in uuid:get-node-id in
   that it sets bit 0 of the the LS-Byte of a 48bit integer with:

     (setf node (dpb #b01 (byte 8 0) (random #xffffffffffff *random-state-uuid*)))

   Apparently, there is some confusion around the RFC's reference to the
   unicast/multicast bit, instead of the arguably more correct local/global bit.
   
   As it is now, when using Tzonev's uuid one can not reliably inspect a v1 UUID
   for its version because the bits are in the wrong sequence and disambiguation
   of of the various v1, v2, v3, v4, and v5 UUIDs is impossible.
   
  :SEE unicly/unicly-compat.lisp for additional details/discussion.

   We could attempt to accommodate this and propagate the error onward or do the
   prudent thing and simply rely on v3, v4, v5 UUIDs instead.

Examples of Common Lisp libraries which make use of UUIDs:

(URL `https://raw.github.com/kraison/vivace-graph-v2/master/triples.lisp')
(URL `https://raw.github.com/lisp/de.setf.resource/master/resource-object.lisp')
(URL `https://raw.github.com/dto/blocky/master/prototypes.lisp')
(URL `https://raw.github.com/fons/cl-mongo/master/src/bson-oid.lisp')
(URL `git://github.com/kraison/kyoto-persistence.git')

An Emacs lisp implementation of RFC 4122 UUID generation:

(URL `https://github.com/kanru/uuid-el')

RFC 4122:

(URL `http://www.ietf.org/rfc/rfc4122.txt')
(URL `http://tools.ietf.org/pdf/rfc4122')


;;; ==============================
;;; EOF


