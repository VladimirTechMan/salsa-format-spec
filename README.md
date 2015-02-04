SALSA Format 0.7 Specification
==============================

**Author: Vladimir "*VladimirTechMan*" Beloborodov**, \<VladimirTechMan@gmail.com\>

##*Abstract*

*This document is a description of Simple Application-Level-Signaling Archive (SALSA) format that can be used to easily log, annotate, exchange and process packet flows of different text-based signaling protocols that are used in modern web or VoIP applications. The need in such a format was largely inspired by the proliferation of WebRTC-based solutions. Yet, it is not limited to WebRTC by any means. For example, this same format can also be useful in the area of traditional SIP-based telephony.*

##1 Motivation behind proposing the SALSA format

Network traces can be collected with tools like Wireshark and saved in such file formats as PCAP or PcapNG. That gives engineers a very precise and detailed picture about what was going on at all levels of the networking stack, up from the data link layer, at a specific network point and during a specific period of time. Unfortunately, in many cases such a deep level of underlying protocol details and the binary formats used to store them make handling the captured flows of application-level signaling protocols unnecessarily complex or even impossible. Some major issues around that are as follows:

* Existing libraries (especially open-source ones) that are intended to read PCAP and similar file formats often fail to correctly handle the TCP message framing and cannot properly re-assemble the application-level signaling protocol packets.
* The traffic coming between the endpoints is typically encrypted these days. Thus, trying to capture details somewhere outside the actual applications that send and receive data will often not allow to look at the actual signaling packets being exchanged over such an encrypted channel.
* In the endpoint applications: Capturing into real PCAP or similar packet capture formats may be not possible and usually is not feasible. That is even more applicable to web browsers and web applications, as well as to mobile applications.

Some of the modern communication libraries and frameworks do provide embedded logging capabilities so that they can log the details of signaling packets being exchanged by the applications. But those packet details are often mixed with some general debug or information printouts. And those logs are in different custom, incompatible formats. Thus, universally handling them in third-party applications is problematic.

With that — and with some other practical needs — in mind, the Simple Application-Level-Signaling Archive (SALSA) format is proposed. It is a simple, concise JSON-based format that aims at being easy to create, easy to annotate, easy to parse and process the logged signaling packet data. Using it, developers and companies can quickly create different types of handy engineering tools, like those for call-flow visualizations or automatic test script generation. And such tools, handling specific signaling protocols, can now become much more compatible with each other and with the signaling libraries and components of client-side and server-side communication applications.

The SALSA format is agnostic to the types of the signaling protocols that it contains. It may be used to archive call/session signaling, media signaling, or domain-specific signaling types (for example, signaling payloads used for monitoring or control of remote objects) — and the format can contain multiple types of signaling protocols at once.

###1.1 Why not HAR?

The HTTP Archive (HAR) format is, indeed, somewhat similar to SALSA. Actually, SALSA was greatly inspired by HAR. But HAR was created largely with the HTTP specifics in mind. And it is tailored for HTTP, and not so much towards capturing and handling traces of arbitrary text-based signaling protocols used by VoIP and web-applications. This is why SALSA is proposed. (As a quick note: It is very easy to combine the "log" root entry of the HAR format and the "salsa" root entry of the SALSA format inside one parent JSON object, shall that be practically useful.)

##2 Conformance requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in the normative parts of this document are to be interpreted as described in RFC 2119.

Requirements phrased in the imperative as part of algorithms (such as "strip any leading space characters" or "return false and abort these steps") are to be interpreted with the meaning of the key word ("must", "should", "may", etc) used in introducing the algorithm.

Some conformance requirements are phrased as requirements on attributes, methods or objects. Such requirements are to be interpreted as requirements on user agents.

Conformance requirements phrased as algorithms or specific steps may be implemented in any manner, so long as the end result is equivalent.

##3 Terminology and Definitions

* The construction "a Foo object", where Foo is actually an interface, is sometimes used instead of the more accurate "an object implementing the interface Foo".

* __SALSA file creator.__ An application or a software component (library, module, web service, etc.) that produces a ready file conformant with all the requirement of this specification. Please, note that a SALSA file creator is not necessarily doing the actual packet logging activities. For example, it may instead be a tool convering existing packet capture files (represented, for example, in a binary format like PCAP or PcapNG) into the SALSA format. It may also be a tool to re-format, or merge together, or filter existing files in the SALSA format.

* __Packet logger.__ An application or a software component (library, module, web service, etc.) that does the actual logging (either complete capturing and sniffing, or just "dumping") of the network packets to be, eventually, archived in a file using the SALSA format. Please, note that a packet logger does not necessarily create ready files in the SALSA format. It may instead save the results in some other format that a SALSA file creator will later read and then create a SALSA-formatted file based on it. It may also be directly passing the packets being logged to a SALSA file creator for further processing (for example, using module APIs inside one application, or over some system-level or network-level connection mechanisms).

* __SALSA file consumer.__ An application or a software component (library, module, web service, etc.) that is able to read files in the SALSA format and then handle, in some way, the information contained in those files. Typically, a SALSA file consumer does not have the functions of SALSA file creator or packet logger. But in specific cases it may include some of them too (for example, think of a utility to re-format, merge together, or filter existing SALSA files).

* __UTC.__ Coordinated Universal Time as maintained by the Bureau International des Poids et Mesures (BIPM).

* The construction "SALSA file" is used throughout this document in the sense "a file properly created and formatted according to the SALSA specification".

##4 The SALSA format

The SALSA format is based on JSON, as described in RFC 4627.

###4.1 Encoding

A SALSA file is REQUIRED to be saved in UTF-8 encoding. Other encodings are forbidden. A reader MUST ignore a byte-order mark if it exists in the file, and a writer MAY emit a byte-order mark in the file.

###4.2 List of objects

####4.2.1 salsa

This object represents the root of the exported data. This object MUST be present and its name MUST be "salsa". The object contains the following name/value pairs:

  JSON Name  |JSON Type| Description
 ------------|---------|------------
 "version"   | string  | Required. Version number of the SALSA format used for the given file.
 "creator"   | object  | Optional. An object of type creator that contains the information about SALSA file creator.
 "startedDateTime" | string | Optional. The timestamp of the moment when the packet logging activity actually started. The value is formatted according to a subset of formats defined in ISO 8601 (*see details below*).
 "duration"  | string  | Optional. Total duration of the packet logging activity, in seconds.
 "geolocation" | object | Optional. The geographical position where the packets were logged by packet logger. 
 "comment"   | string  | Optional. A comment provided by the user or the application.
 "protocol"  | string  | Optional. The name of the signaling protocol being represented by all the entries in the "packets" array.
 "transport" | string  | Optional. The name of the transport being used to exchange all the packets represented by the "packets" array.
 "packets"   | array   | Required. An array of objects of type packet, where each object represents a logged signaling protocol packet.

####*4.2.1.1 "startedDateTime"*

A SALSA file creator SHOULD provide the "startedDateTime" value. For a specific SALSA-formatted file, the "startedDateTime" string value MUST represent the moment when the packet logging activity started. (Note that in the case of converting existing packet capture files to the SALSA format, the timestamps MUST be for when the packets in those source files where logged, not for the time frame when the conversion was performed itself.) In the simplest case, that moment is equal to when the first packet was actually logged by the application. But it may also be earlier on the timeline, for example, if the application was waiting for some time before it actually received and logged the first signaling packet.

Whenever the "startedDateTime" value is provided in the file, all the individual "time" values of "packet" objects, in the "packets" array, MUST be relative to that "startedDateTime" value. When the "startedDateTime" value cannot be known, the SALSA file creator MAY still provide the "time" values for "packet" objects relative to the moment when the packet logging was actually started (for example, when converting to the SALSA format from an existing capture file in a format that provides that relative timing information in it, but not the absolute start time of the logging activity). If the SALSA file creator has no possibility to determine the time stamps of individual packets relative to the moment when the logging activity was started, then it MUST assume that moment to be equal to the moment when the first packet was actually logged (and, thus, the "time" value of that first packet object in the "packets" array will be zero).

The "startedDateTime" string value, when provided, MUST be formatted in accordance with the ISO 8601 format for combined date and time, with explicit delimiters between all the date and time unit components, and it MUST include seconds as its smallest time unit provided. The seconds value SHOULD also include, at least, thousandth fractions, to support the millisecond precision. It it RECOMMENDED to provide extra digits to the fractional part, to support a sub-millisecond precision, when that is possible. The SALSA file creator MAY add the time zone information, in accordance to ISO 8601 specifications, when it is directly available or can be easily determined by the file creator. Otherwise, the "startedDateTime" string value specifies the local time on the system where the packets were logged. When indicating a time zone, in a file compliant with this version of SALSA format spec, the offset from UTC MUST be specified in one of the following forms, as appropriate: "+hh:mm", "-hh:mm", or "Z". (The "Z" value is used in ISO 8601 as a short notation for the zero UTC offset and is semantically equivalent to "+00:00".) Thus, the format of "startedDateTime" string value MUST be "YYYY-MM-DDThh:mm:ss.sss" when representing a local time (for example, "2013-10-22T17:18:30.457") and it MUST be "YYYY-MM-DDThh:mm:ss.sssTZD" when representing a global time, relative to UTC (for example, "2013-10-22T17:18:30.457+03:00" or "2013-10-22T14:18:30.457Z"; note that these two example values represent the same global time). Please, note that, according to the ISO 8601 specification, value "-00:00" is not allowed and thus it MUST NOT be used in SALSA-formatted files. Other possible formats declared in ISO 8601 for specifying time zones MUST NOT be used in SALSA. (That and other restrictions given above on the format of "startedDateTime" are for the sake of easier parsing and formatting SALSA files.)

The usage of ISO 8601-compliant format, as opposed to a numeric value equal to the absolute clock time on the system, is intentional in SALSA: It allows for more compatibility between different systems when doing some calendar calculations, even basic ones. (The absolute time values on a system are normally relative to some predefined epoch and different systems may have different epoch "starting points". Thus, a more portable approach is beneficial.) Also, the possible string formats specified above are much easier to understand for a person checking a SALSA-formatted file with a plain text editor.

Please, note that a global time point can be correctly represented using any of the available time zone offsets. For example, many platforms allow to easily get an ISO-formatted time value that is compliant with the above requirements and is adjusted to UTC (that is, it has a zero time zone offset). Thus, the SALSA file consumer SHOULD NOT use the time offset information as an evidence of the geographical location (an area or region) where the packet logger performed its logging activity. Instead, when the geolocation information is available and the SALSA file creator is allowed to provide it in the file, the geographical position of the packet logger MUST always be conveyed by using the "geolocation" object.

####*4.2.1.2 "duration"*

The "duration" value, if added by the SALSA file creator, MUST be equal to the total duration, in seconds, of the packet logging activity (starting from the moment represented by the "startedDateTime" value). The numeric equivalent of the "duration" string value MUST be equal to or greater than the numeric equivalent of the "time" string value of the last "packet" object in the "packets" array.

The SALSA file creator SHOULD handle and output all the duration values, at least, up to their thousandth fractions, to support the millisecond precision. It it RECOMMENDED to provide extra digits to the fractional part, to support a sub-millisecond precision, when that is possible. The value of "duration" MUST be a string representation of the numeric value of logging activity duration and it MUST only contain digits (0-9) and, optionally, one dot character (.) to delimit the integer part and the fractional part of the numeric value. (For the explanation on utilizing the JSON string type versus the number type, to represent relative timing values in SALSA, please, refer to section 4.2.3, to the part discussing the "time" value of the "packet" object.)

Providing the "duration" value is RECOMMENDED in the cases where the packet logging activity did not immediately finish at the moment when the last packet (represented in the "packets" array) was logged and the fact that there was no other signaling packets between that last logged packet and the actual end of the packet logging activity can be useful, as an extra piece of information, to the SALSA file consumers.

####*4.2.1.3 "transport" and "protocol"*

For the sake of compatibility, all string values for "protocol" and "transport" MUST be in the lower case. For specific requirements on naming the "protocol" entries, please, refer to section 4.3.

For the "transport" entry, the following values are suggested in this version of the SALSA format: "udp", "tcp", "websocket", "webrtc-datachannel". Please, note that in this case the transport name is not necessarily limited to the actual transport protocol being used (as "udp" and "tcp" are, for example), but may also reflect the actual mechanism being used to exchange the signaling packets (for instance, "websocket", which is built on top of TCP, or "webrtc-datachannel", which is essentially SCTP over UDP). (Going forward, more values may be added. For example, it is not clear yet if adding "tls" and "dtls", to augment more general "tcp" and "udp" transports, would be beneficial for practical purposes of SALSA.)

Providing the "protocol" entry in the "salsa" object is RECOMMENDED in case if all the packets represented by objects inside the "packets" array belong to the same signaling protocol. Providing the "transport" entry in the "salsa" object is RECOMMENDED in case if all the packets represented by objects inside the "packets" array are exchanged over the same transport mechanism. When the "protocol" entry or the "transport" entry in the "salsa" object is provided, the SALSA file creator SHOULD NOT provide the corresponding individual "protocol" or "transport" entries in the "packet" objects inside the "packets" array.

The SALSA file creator SHOULD provide the values of "transport" and "protocol" entries whenever that information is readily available to it. When a SALSA file creator cannot identify the actual transport mechanism or the actual signaling protocol being used, or in case where identifying them would require taking additional steps that would notably affect the expected responsiveness characteristics of the application, the SALSA file creator MAY opt for not providing that information in the file.

####4.2.2 creator

This object contains information about the SALSA file creator and it contains the following name/value pairs:

  JSON Name|JSON Type| Description
 ----------|---------|------------
 "name"    | string  | Required. The name of the SALSA file creator that produced the given SALSA file.
 "version" | string  | Required. The version number of the SALSA file creator that produced the given SALSA file.
 "comment" | string  | Optional. A comment provided by the user or the application.

####4.2.3 geolocation

This object contains information about the geographical position where the packets represented in the given SALSA file were originally collected by packet logger.

  JSON Name  |JSON Type| Description
 ------------|---------|------------
 "latitude"  | number  | Required. The latitude of the position, in decimal degrees.
 "longitude" | number  | Required. The longitude of the position, in decimal degrees.
 "accuracy"  | number  | Required. The accuracy of position with latitude and longitude values.
 "altitude"  | number  | Optional. The altitude of the position, in meters above the mean sea level.
 "altitudeAccuracy" | number  | Optional. The altitude accuracy of position, in meters.

All values of "accuracy" and "altitudeAccuracy" MUST be non-negative real numbers.

####4.2.4 packets

The "packets" array represents a time-ordered sequence of the logged signaling packets. It has the following name/value pairs:

  JSON Name  |JSON Type| Description
 ------------|---------|------------
 "time"      | string  | Required. Packet logging time, in seconds, relative to the moment when the packet logging activity was started.
 "protocol"  | string  | Optional. The name of the signaling protocol of the packet.
 "transport" | string  | Optional. The name of the transport that was used to transmit the packet.
 "src"       | object  | Required. The identification details of the source (sender) of the packet.
 "dst"       | object  | Required. The identification details of the destination (receiver) of the packet.
 "format"    | string  | Optional. The format of the "body" entry in the current "packet" object. If omitted, "plain-text" MUST be assumed.
 "body"      | *depending on "format" specified* | Required. The actual data of archived signaling packet, represented according to the specified "format".
 "comment"   | string  | Optional. A comment provided by the user or the application about the packet itself or about the part of interaction between source and destination that the packet is used for.

The "packet" objects inside the "packets" array MUST be in the ascending order, according to the numerical equivalents of "time" string values in these objects.

####*4.2.4.1 "time"*

The "time" value in a "packet" object MUST be calculated relative to the moment when the logging activity was started. That way, when the SALSA file creator provides the "startedDateTime" value in the "salsa" object, the SALSA file consumers can easily calculate the absolute timestamps of individual packets, if required. On the other hand, even if the "startedDateTime" value is not provided in the file (for example, because it was not available from the original packet capture format that was converted to the SALSA format), the relative values will still be available and will give the information necessary to analyze relative timing characteristics.

The "time" values MUST be in seconds. The SALSA file creator SHOULD handle and output all the time values, at least, up to their thousandth fractions, to support the millisecond precision. It it RECOMMENDED to provide extra digits to the fractional part, to support a sub-millisecond precision, when that is possible. The "time" string value MUST only contain digits (0-9) and, optionally, one dot character (.) to delimit the integer part and the fractional part of the numeric value.

The SALSA format uses a string representation of timestamps, rather than the number type available in the JSON format. That is intentional, for two reasons:
* It allows for better control over the formatting of the timestamps with a millisecond or sub-millisecond precision (that is, with a fractional part in them): Some of the available libraries that read and write the JSON format tend to support only a fixed (and non-configurable) maximum number of digits in a fractional part, which may be insufficient for parsing or saving the timestamps in SALSA file creators or consumers based on those libraries.
* It allows for more flexibility in dealing with longer integer and floating-point numeric values on the architectures and in some (typically, older) programming languages where the basic numeric type(s) do not provide a sufficient value range to hold such numeric values in them without a representation error.

####*4.2.4.2 "transport" and "protocol"*

The SALSA file creator SHOULD specify a "protocol" value for each "packet" object, unless it has specified the "protocol" value in the "salsa" object (which signals that all the packets represent the same protocol). When the "protocol" value in the "salsa" object is specified, the SALSA file creator SHOULD NOT specify the "protocol" value for each packet.

For specific requirements on naming the "protocol" entries, please, refer to section 4.3.

The SALSA file creator SHOULD specify a "transport" value for each "packet" object, unless it has specified the "transport" value in the "salsa" object (which signals that all the packets are transmitted over the same transport mechanism). When the "transport" value in the "salsa" object is specified, the SALSA file creator SHOULD NOT specify the "transport" value for each packet.

####*4.2.4.3 "format" and "body"*

The "format" string is optional. In this version of SALSA, three values are officially specified: "plain-text", "plain-text-multipart" and "base64". When the "format" string value is not specified for a "packet" object, the "plain-text" value MUST be assumed by default.

When the "format" parameter value is "plain-text": The "body" value of a "packet" object MUST be a string. The string representation contains the actual signaling packet data (text in UTF-8) that were sent and received (over the transport mechanism specified with the "transport" string value, if provided).

When the "format" parameter value is "plain-text-multipart": The "body" value of a "packet" object MUST be an array of strings. In this case the original packet is recreated as simple concatenation of all the strings in the array, in the given order. The "plain-text-multipart" format allows to represent signaling packets of many modern text-based signaling protocols, like SIP, XMPP or JSON, in a more structured manner that can be easier for a human to read and analyze directly from the file (using a plain text editor, or alike). For a more compact representation, using the "plain-text" format is RECOMMENDED.

When the "format" parameter value is "base64": The "body" value of a "packet" object MUST be a string. In this case, the string representation contains the actual signaling packet data encoded using the base64 encoding, as described in RFC 4648. All SALSA file creators and consumers MUST comply with the requirements of RFC 4648 when encoding and decoding signaling packet bodies for the "base64" format option.

When representing packets of a specific signaling protocol, the usage of the described format options is as follows:
* When the signaling protocol is known to the SALSA file creator and it is a text-based signaling protocol that mandates the usage of the UTF-8 encoding, the SALSA file creator MUST use one of the following "format" value options: "plain-text" or "plain-text-multipart".
* When the signaling protocol is known to the SALSA file creator and it is a text-based signaling protocol that allows using both the UTF-8-compliant encodings and text encodings non-compliant with UTF-8. Then the SALSA file creator MAY identify the actual text encoding of the packet under consideration (for example, based on a parameter or attribute in the packet format that explicitly specifies the text encoding used in that packet). The SALSA file creator MUST use the "plain-text" or "plain-text-multipart" format options for all encodings compliant with UTF-8. For all other encodings, the SALSA file creator MUST use the "base64" format option. When the SALSA file creator is not able to identify the actual text encoding of the packet, or the packet specifies a text encoding name that is not known to the SALSA file creator, or identifying the text encoding of the packet would notably affect the expected performance characteristics of the application, the SALSA file creator MUST use the "base64" format option.
* In all other cases, the SALSA file creator MUST use the "base64" format option.

When using the "plain-text" or "plain-text-multipart" format options, the SALSA file creator MUST ensure that all the string values representing packet bodies are valid in terms of the UTF-8 encoding scheme. Where the SALSA file creator is a part of the appliation, or it uses other modules or components, that already do this check (for example, when the application is the actual source or the final destination of the packets being logged), the SALSA file creator MAY rely on those known checks as a matter of validating the strings representing packet bodies.

####4.2.5 src and dst

The "src" object identifies the source (sender) of signaling protocol packet. The "dst" object identifies the destination (receiver) of signaling protocol packet. They both have the same format, with the following name/value pairs:

  JSON Name|JSON Type| Description
 ----------|---------|------------
 "name"    | string  | Required. The unique name of the signaling packet sender or receiver.
 "ipaddr"  | string  | Optional. The IP address of the signaling packet sender or receiver.
 "port"    | number  | Optional. The TCP/UDP/SCTP port of the signaling packet sender or receiver.

The "name" string value is an arbitrary symbolic name, encoded in UTF-8, and it MUST be present inside each "src" and "dst" object. It uniquely idenitifies each source and desitnation peer referenced in a SALSA file. It MAY also serve the purpose of better documenting/annotating the source and destination peers referenced in a SALSA file by using a logical naming scheme that is clear and easier to understand to the people using the file.

The same unique "name" value MUST consistently be used to reference the same sender or the same receiver multiple times in a SALSA file. Different senders and receivers (for example, those having different IP addresses or different ports on the same IP address) MUST get different names when they are represented in a SALSA file. Note that the users of SALSA file MAY provide better naming annotations at any later point in time, by replacing any of the original (unique) names with a more descriptive (unique) one.

It is RECOMMENDED that the "ipaddr" and "port" values are provided in the SALSA file whenever the SALSA file creator has, or can easily obtain, those two values, respectively.

The "ipaddr" string value MUST be formatted according to the standard dotted decimal representation for IPv4 addresses, and according to the string formats recommended in RFC 5952 for IPv6 addresses.

The "port" number value SHOULD be provided by the SALSA file creator whenever it is aware which specific ports are used by the source (sender) and the destination (receiver) of the signaling packet. When the SALSA file creator cannot identify the actual port number for the source or for the destination, or when identifying it would require taking additional steps that would notably affect the expected responsiveness characteristics of the application, the SALSA file creator MAY opt for not providing that specific port number in the file. The "port" value MUST be a positive integer numeric value.

In the case where several specific ports on the same IP address are consistently used to exchange specific types of signaling: If the SALSA file creator is not able to identify the "port" value, or it opts for not providing that information due to performance considerations, as described above, and yet the SALSA file creator is able to distinguish between the specific types of signaling or signaling usage on different ports, it is RECOMMENDED that the SALSA file creator provides and consistently applies different (meaningful) names for those different signaling channels (signaling usages) that are found at the same IP address.

If the "ipaddr" and "port" values are available, and no better naming scheme is possible, the "name" string value SHOULD be a combination of the "ipaddr" and "port" values with an appropriate formatting. For example, the name MAY be formatted like "192.168.34.17:5070" in case of IPv4 and like "[1fff:0:a88:85a3::ac1f]:80" in case of IPv6.

###4.3 Specifying protocol names in SALSA

The string values for the "protocol" entry, as specified in sections 4.2.1 and 4.2.3 (for the "salsa" and "packet" objects), MUST all be in the lower case.

To achieve better interoperability in handling SALSA files, the following regulations are effective:
* When the protocol is standardized and, thus, it has an established standard protocol name, then SALSA file creators and consumers SHOULD use this name as the string value. (For example, "sip", "xmpp", "json", "http", "sdp", etc.)
* When the protocol is not standardized and the protocol format is unique (that is, it is not based on any other existing signaling protocol, and it is not a variation, or an extension, or a specialization of an exiting protocol, or it is such a modification of the existing protocol that makes it syntactically incompatible with the original one), then the SALSA file creators and consumers aware of that specific protocol MAY use the currently established (maybe, unofficial or internal) protocol name "as is". Note that in this case, often, only these SALSA file creators and consumers will be able to properly handle that protocol's packets in SALSA files. On the other hand, some general tools and services, like those drawing call-flow diagrams, MAY still provide a reasonable handling and representation for the communications using such unknown protocols.
* When the protocol itself is not standardized, but it is based on, or is a variation (extension, specialization) of an existing protocol (whether a standardized one or not) and is backward-compatible with that existing protocol, at least, syntactially. Then it is RECOMMENDED that the SALSA file creators and consumers aware of that specific protocol use an extended protocol naming scheme approach, as follows: "\<base protocol name\>-\<name of custom protocol or specialization\>-\<name of even more custom protocol or specialization\>". The same approach is RECOMMENDED even if the protocol *is* a standardized extension or a variation of another protocol (see some examples on that below).

As an example for the last item in the regulations above, consider a company that uses their own, specific, custom signaling protocol that is based on the JSON format (protocol). Let them call their custom protocol "proFoo". In this case, the recommendation is to avoid using just the "profoo" name when archiving packets of that protocol with SALSA and, instead, to use the "json-profoo" name. That way the SALSA file consumers, even if they are not aware of specific details of the "proFoo" protocol can have a better cue that its basic format is JSON and try to handle the packets of that protocol more specifically (for example, to format and visually represent the packet body contents in the JSON format). While it can definitely be possible to create a number of heuristics for a SALSA file consumer to attempt to intellegently decide on the basic formats and the protocols of individual packets with unknown protocol names in a SALSA file. Still, the naming scheme described above can greatly reduce the complexity and processing time as SALSA file consumers can rely on the suggestions provided with this naming approach.

It is also worth noting that some of the existing specialized signaling protocols already fit into this convention quite naturally. As an example, consider the two common SIP-based protocols for SIP-ISUP interworking, called "SIP-I" and "SIP-T". Shall a SALSA file creator need to archive packets of those protocols, it can use the "sip-i" and "sip-t" protocol names, respectively, in the file. As another example, consider the XML-based and JSON-based approaches to the remote procedure call (RPC) signaling, often named "XML-RPC" and "JSON-RPC". When archiving packets from those RPC protocols with SALSA, using "xml-rpc" and "json-rpc" names would be a recommended approach.

When a SALSA file consumer cannot identify even the basic signaling protocol (format) for an archived packet, it MAY skip the packet. In this case it SHOULD provide an indication or a notification to the end user(s) on the failed attempt to handle that packet. It MAY also ask the user if processing this SALSA file need to be continued. Depending on its funcational scope, the SALSA file consumer MAY also try to provide a reasonable high-level handling of a packet for an unknown protocol name (for instance, based on its source and destination details) — again, as an example, consider a call-flow visualization tool: It can just draw an approprite call flow arrow between the source and destination on the diagram and annotate it for the end user with the given protocol name.

Finally, as more tools get support of the SALSA format in them, some of them may offer a more modular approach, where the companies or communities behind certain custom signaling protocols will be able to supply 3rd-party add-ons or extension modules that will enable the tools to recognize and handle those custom protocols in details.

##5 Versioning Scheme

The spec number has following syntax:  
\<major-version-number\>.\<minor-version-number\>

Where the major version indicates overall backwards compatibility and the minor version indicates incremental changes. So, any backwardly compatible changes to the spec will result in an increase of the minor version. If an existing fields had to be broken then major version would increase (e.g. 2.0).

Examples:  
1.2 -> 1.3  
1.109 -> 1.110 (in case of 109 more changes)  
1.5 -> 2.0 (2.0 is not compatible with 1.5)  

##6 Privacy

The SALSA format may contain privacy and security sensitive data and in such a case the SALSA file creator SHOULD find some way to notify the end user of this fact before the file is transferred to anyone else.

##7 References

[IETF RFC 2119]  
    Key words for use in RFCs to Indicate Requirement Levels, Scott Bradner, Author. Internet Engineering Task Force, March 1997. Available at http://www.ietf.org/rfc/rfc2119.txt.  
[IETF RFC 4627]  
    The application/json Media Type for JavaScript Object Notation (JSON), D. Crockford, Author. Internet Engineering Task Force, July 2006. Available at http://www.ietf.org/rfc/rfc4627.txt.  
[IETF RFC 5952]  
    A Recommendation for IPv6 Address Text Representation, S. Kawamura, M. Kawashima, Authors. Internet Engineering Task Force, August 2010. Available at http://www.ietf.org/rfc/rfc5952.txt.  
[IETF RFC 4648]  
    The Base16, Base32, and Base64 Data Encodings, S. Josefsson, Author. Internet Engineering Task Force, October 2006. Available at http://www.ietf.org/rfc/rfc4648.txt.  
[HAR]  
    HTTP Archive (HAR) format, Jan Odvarko, Arvind Jain, Andy Davies, Editors. World Wide Web Consortium, August 14, 2012. The latest editor's draft is available at http://dvcs.w3.org/hg/webperf/raw-file/tip/specs/HAR/Overview.html.  
