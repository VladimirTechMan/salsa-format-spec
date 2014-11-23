ALSA Format 0.1 Specification
=============================

Author: Vladimir "VladimirTechMan" Beloborodov, \<VladimirTechMan@gmail.com\>

*This document is a description of Application-Level Signaling Archive (ALSA) format that can be used to easily capture, annotate, exchange and process packet flows of different text-based signaling protocols that are used in modern web or VoIP applications. The need in such a format was largely inspired by the proliferation of WebRTC-based solutions. Yet, it is not limited to WebRTC by any means. For example, this same format can also be useful in the area of traditional SIP-based telephony.*

##1 Motivation behind proposing the ALSA format

Network traces can be collected with tools like Wireshark and saved in such file formats as pcap or pcapng. That gives engineers a very precise and detailed picture about what was going on at all levels of the networking stack, up from the data link layer, at a specific network point and during a specific period of time. Unfortunately, in many cases such a deep level of underlying protocol details and the binary formats used to store them make handling the captured flows of application-level signaling protocols unnecessarily complex or even impossible. Some major issues around that are as follows:

* Existing libraries (especially open-source ones) that are intended to read pcap and similar file formats often fail to correctly handle the TCP message framing and cannot properly re-assemble the application-level signaling protocol packets.
* The traffic coming between the endpoints is typically encrypted these days. Thus, trying to capture details somewhere outside the actual applications that send and receive data will often not allow to look at the actual signaling packets being exchanged over such an encrypted channel.
* In the endpoint applications: Capturing real pcap or similar trace formats may be not possible and usually is not feasible. That is even more applicable to web browsers and web applications as well as mobile applications.

Some of the modern communication libraries and frameworks do provide embedded logging capabilities so that they can capture the details of signaling packets being exchanged by the applications. But those packet details are often mixed with some general debug or information printouts. And those logs are in different custom, incompatible formats. Thus, universally handling them in third-party applications is problematic.

With that — and with some other practical needs — in mind, the Application-Level Signaling Archive (ALSA) format is proposed. It is a simple, concise JSON-based format that aims at being easy to create, easy to annotate, easy to parse and process the captured signaling packet data. Using it, developers and companies can quickly create different types of handy engineering tools, like those for call-flow visualizations or automatic test script generation. And such tools, handling specific signaling protocols, can now become much more compatible with each other and with the signaling libraries and components of client- and server-side communication applications.

###1.1 Why not HAR?

The HTTP Archive (HAR) format is, indeed, somewhat similar to ALSA. Actually, ALSA was greatly inspired by HAR. But HAR was created largely with the HTTP specifics in mind. And it is tailored for HTTP, and not so much towards capturing and handling traces of arbitrary text-based signaling protocols used by VoIP and web-applications. This is why ALSA is proposed. (As a quick note: It is very easy to combine the "log" root entry of the HAR format and the "alsa" root entry of the ALSA format inside one parent JSON object, shall that be practically useful.)

##2 Conformance requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in the normative parts of this document are to be interpreted as described in RFC 2119.

Requirements phrased in the imperative as part of algorithms (such as "strip any leading space characters" or "return false and abort these steps") are to be interpreted with the meaning of the key word ("must", "should", "may", etc) used in introducing the algorithm.

Some conformance requirements are phrased as requirements on attributes, methods or objects. Such requirements are to be interpreted as requirements on user agents.

Conformance requirements phrased as algorithms or specific steps may be implemented in any manner, so long as the end result is equivalent.

##3 Terminology

The construction "a Foo object", where Foo is actually an interface, is sometimes used instead of the more accurate "an object implementing the interface Foo".

##4 The ALSA format

The ALSA format is based on JSON, as described in RFC 4627.

###4.1 Encoding

An ALSA file is REQUIRED to be saved in UTF-8 encoding. Other encodings are forbidden. A reader MUST ignore a byte-order mark if it exists in the file, and a writer MAY emit a byte-order mark in the file.

###4.2 List of objects

####4.2.1 ALSA

This object represents the root of the exported data. This object MUST be present and its name MUST be "alsa". The object contains the following name/value pairs:

  JSON Name  |JSON Type| Description
 ------------|---------|------------
 "version"   | string  | Required. Version number of the format.
 "creator"   | object  | Optional. An object of type creator that contains the name and version information of the log creator application.
 "comment"   | string  | Optional. A comment provided by the user or the application.
 "protocol"  | string  | Optional. The name of the signaling protocol being represented by all the entries in the "packets" array.
 "transport" | string  | Optional. The name of the transport being used to exchange all the packets represented by the "packets" array.
 "packets"   | array   | Required. An array of objects of type packet, each representing one exported (captured) signaling protocol packet.

For the sake of compatibility, all string values for "protocol" and "transport" are given in lower case. Using standard protocol names (or the commonly established protocol names, if they are non-standard yet) is REQUIRED. For example, "sip", "xmpp", "jingle", "json", "http", "sdp", etc.

For the "transport" entry, the following values are suggested in this version of the ALSA format: "udp", "tcp", "websocket", "webrtc-datachannel". Please, note that in this case the transport name is not necessarily limited to the actual transport protocol being used (as "udp" and "tcp" are, for example), but may also reflect the actual mechanism being used to exchange the signaling packets (for instance, "websocket", which is built on top of TCP, or "webrtc-datachannel", which is essentially SCTP over UDP). Going forward, more values may be added. (For example, it is not clear yet if adding "tls" and "dtls", in addition to general "tcp" and "udp" transports, would be beneficial for practical purposes of ALSA.)

Providing the "protocol" entry in the "alsa" object is RECOMMENDED in case if all the packets represented by objects inside the "packets" array belong to the same signaling protocol. Providing the "transport" entry in the "alsa" object is RECOMMENDED in case if all the packets represented by objects inside the "packets" array are exchanged over the same transport mechanism. When the "protocol" entry or the "transport" entry in the "alsa" object is provided, the ALSA creator application SHOULD NOT provide the corresponding individual "protocol" or "transport" entries in the "packet" objects inside the "packets" array.

The ALSA creator application SHOULD provide the values of "transport" and "protocol" entries whenever that information is readily available to it. When the application cannot identify the actual transport mechanism or the actual signaling protocol being used, or in case where identifying them would require taking additional steps that would notably affect the expected responsiveness characteristics of the application, the application MAY opt for not providing that information in the ALSA format.

####4.2.2 creator

This object contains information about the ALSA creator application (framework, module, etc.) and contains the following name/value pairs:

  JSON Name|JSON Type| Description
 ----------|---------|------------
 "name"    | string  | Required. The name of the application that created the log.
 "version" | string  | Required. The version number of the application that created the log.
 "comment" | string  | Optional. A comment provided by the user or the application.

####4.2.3 packets

The "packets" object represents an ordered sequence of the captured signaling packets. It has the following name/value pairs:

  JSON Name  |JSON Type| Description
 ------------|---------|------------
 "time"      | string  | Required. Packet capture time, in milliseconds, formatted as a string. (A fractional part for sub-millisecond precision is allowed.)
 "protocol"  | string  | Optional. The name of the signaling protocol of the packet.
 "transport" | string  | Optional. The name of the transport being used to transmit the packet.
 "src"       | object  | Required. The identification details of the source (sender) of the packet.
 "dst"       | object  | Required. The identification details of the destination (receiver) of the packet.
 "format"    | string  | Optional. The format of the "body" entry in the current "packet" object. If omitted, "plain-text" is assumed.
 "body"      | string or array | Required. The actual data of captured signaling packet, according to the "format" string.
 "comment"   | string  | Optional. A comment provided by the user or the application about the packet.

The "packet" objects inside the "packets" array MUST be in the ascending order, according to the numerical equivalents of "time" string values in these objects.

The "time" values MUST be in milliseconds. The ALSA creator application MAY add a fractional part to those values to provide a sub-millisecond precision, when that is possible.

The "time" string value MUST only contain digits and (optionally) one dot character (.) to delimit the integer part and the fractional part of the numeric value.

The ALSA format uses a string representation of timestamps, rather than the number type available in the JSON format. That is intentional, for two reasons:
* It allows a better control over the formatting of the timestamps with a sub-millisecond precision (that is, with a fractional part in them): Some of the available libraries that read and write the JSON format tend to support only a fixed (and non-configurable) maximum number of digits in a fractional part, which may be insufficient for parsing or saving the timestamps in the applications that handle the ALSA format based on those libraries.
* It allows more flexibility in dealing with longer integer and floating-point numeric values on the architectures and in some (typically, older) programming languages where the basic numeric type(s) do not provide a sufficient value range to hold such numeric values in them "as is".

When creating a new file in the ALSA format for the signaling packets captured internally, the ALSA creator application SHOULD use the absolute value of the system time clock to set the "time" value for the "packet" object (that is, the timestamp of the moment when the application captured this packet). When creating an ALSA-formatted representation of an existing network trace capture file (for example, of a pcap file), the ALSA creator application SHOULD use the packet timestamp values available inside that original file to provide the equivalent "time" values, in milliseconds, in the ALSA representation, reflecting the absolute timestamps on the system where that original file was captured (for the timeframe when it was captured). If only relative timestamps are available inside the file format being converted, or if it is not clear whether the timestamps in the original file represent the absolute or relative timing values for the system where the packet capture was done, then the ALSA creator application SHOULD just use those available values, appropriately converting them to the millisecond time units for the ALSA format. The creator application MAY add a specific note on that to the "comment" string value of the "alsa" root object, to better document the meaning of "time" values inside the ALSA-formatted file.

The ALSA creator application SHOULD specify a "protocol" value for each "packet" object, unless it has specified the "protocol" value in the "alsa" object (which means that all the packets represent the same protocol). When the "protocol" value in the "alsa" object is specified, the creator application SHOULD NOT specify the "protocol" value for each packet.

The ALSA creator application SHOULD specify a "transport" value for each "packet" object, unless it has specified the "transport" value in the "alsa" object (which means that all the packets are transmitted over the same transport mechanism). When the "transport" value in the "alsa" object is specified, the creator application SHOULD NOT specify the "transport" value for each packet.

The "format" string is optional. In this version of ALSA, only one value is officially specified: "plain-text". And that value is assumed by default when the "format" string value is not specified for a "packet" object.

When the "format" parameter value is "plain-text": The "body" value of a "packet" object can be either a string or an array of strings. The string representation contains the actual signaling packet data (text data) being sent and received (over the transport mechanism specified, if any). If an array of strings is used as the "body" value, then the actual packet can be recreated as a concatenation of those strings, in the given order, where a CRLF pair (which is equal to the "\r\n" JSON-formatted string) is added at the end of each string in the array. That allows to represent packets of many modern signaling protocols, like SIP or XMPP, in a way that can be easier for a human to read and analyze. The single-string representation, on the other hand, allows for smaller ALSA format sizes. (And doing a conversion between the two, if necessary, is quite easy to implement.)

####4.2.4 src and dst

The "src" object identifies the source (sender) of signaling protocol packet. The "dst" object identifies the destination (receiver) of signaling protocol packet. They both have the same format, with the following name/value pairs:

  JSON Name|JSON Type| Description
 ----------|---------|------------
 "ipaddr"  | string  | Required. The IP address of the signaling packet sender or receiver.
 "port"    | number  | Optional. The TCP/UDP/SCTP port of the signaling packet sender or receiver.
 "name"    | string  | Optional. The name of the signaling packet sender or receiver.

The "ipaddr" string value MUST be formatted according to the standard dotted decimal representation for IPv4 addresses, and according to the string formats recommended in RFC 5952 for IPv6 addresses.

The "port" string value SHOULD be provided by the ALSA creator application whenever the application is aware which ports are used by the source (sender) and the destination (receiver) of the signaling packet. When the application cannot identify the actual port number for one of them, or when identifying it would require taking additional steps that would notably affect the expected responsiveness characteristics of the application, the application MAY opt for not providing that specific port number in the ALSA format. The "port" value must be a positive integer numeric value.

The "name" string value serves the purposes of better documenting/annotating the source and destination peers referenced in the ALSA format. It maybe an arbitrary symbolic name. It is REQUIRED that the same name is consistently used to reference the same sender or the same receiver multiple times in the ALSA format. It is RECOMMENDED that different senders and receivers (having different IP addresses or different ports on the same IP address) get different names in the ALSA format.

In the case where several specific ports on the same IP address are consistently used to exchange specific types of signaling: If the ALSA creator application cannot identify the "port" value, or it opts for not providing that information due to performance considerations, as described above, and yet the creator application is able to distinguish between these specific types of signaling on different ports, it is RECOMMENDED that the application provides and consistently applies different (meaningful) names for those different signaling channels (signaling usages) at the same IP address.

When no better naming is possible, the ALSA creator application SHOULD use a name representing the combination of the "ipaddr" and "port" values, with appropriate formatting. For example, "192.168.34.17:5070" in case of IPv4 and "[1fff:0:a88:85a3::ac1f]:80" in case of IPv6. That approach allows the users of ALSA-formatted files to easily provide better naming annotations at any later point in time, by replacing any of the original (unique) names with a more descriptive one. 

##5 Versioning Scheme

The spec number has following syntax:  
\<major-version-number\>.\<minor-version-number\>

Where the major version indicates overall backwards compatibility and the minor version indicates incremental changes. So, any backwardly compatible changes to the spec will result in an increase of the minor version. If an existing fields had to be broken then major version would increase (e.g. 2.0).

Examples:  
1.2 -> 1.3  
1.109 -> 1.110 (in case of 109 more changes)  
1.5 -> 2.0 (2.0 is not compatible with 1.5)  

##6 Privacy

The ALSA format may contain privacy and security sensitive data and in such a case the user agent SHOULD find some way to notify the user of this fact before the file is transferred to anyone else.

##7 References

[IETF RFC 2119]  
    Key words for use in RFCs to Indicate Requirement Levels, Scott Bradner, Author. Internet Engineering Task Force, March 1997. Available at http://www.ietf.org/rfc/rfc2119.txt.  
[IETF RFC 4627]  
    The application/json Media Type for JavaScript Object Notation (JSON), D. Crockford, Author. Internet Engineering Task Force, July 2006. Available at http://www.ietf.org/rfc/rfc4627.txt.  
[IETF RFC 5952]  
    A Recommendation for IPv6 Address Text Representation, S. Kawamura, M. Kawashima, Authors. Internet Engineering Task Force, August 2010. Available at http://www.ietf.org/rfc/rfc5952.txt.  
[HAR]  
    HTTP Archive (HAR) format, Jan Odvarko, Arvind Jain, Andy Davies, Editors. World Wide Web Consortium, August 14, 2012. The latest editor's draft is available at http://dvcs.w3.org/hg/webperf/raw-file/tip/specs/HAR/Overview.html.  
