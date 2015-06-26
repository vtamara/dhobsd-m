---
title: man_5_isakmpd.policy
date: 2011-02-01
tags:
---
ISAKMPD.POLICY(5)         OpenBSD Programmer's Manual        ISAKMPD.POLICY(5)

NAME
     isakmpd.policy - policy configuration file for isakmpd

DESCRIPTION
     isakmpd.policy is the policy configuration file for the isakmpd(8)
     daemon, managing security association and key management for the ipsec(4)
     layer of the kernel's networking stack.  The isakmpd(8) daemon, also
     known as the IKEv1 key management daemon, implements the Internet Key
     Exchange version 1 (IKEv1) protocol.  It follows then that references to
     IKE in this document pertain to IKEv1 only, and not IKEv2.

     isakmpd(8) is used when two systems need to automatically set up a pair
     of Security Associations (SAs) for secure communication using IPsec.
     IKEv1 operates in two stages:

     In the first stage (Main or Identity Protection Mode), the two IKE
     daemons establish a secure link between themselves, fully authenticating
     each other and establishing key material for encrypting/authenticating
     future communications between them.  This step is typically only
     performed once for every pair of IKE daemons.

     In the second stage (also called Quick Mode), the two IKE daemons create
     the pair of SAs for the parties that wish to communicate using IPsec.
     These parties may be the hosts the IKE daemons run on, a host and a
     network behind a firewall, or two networks behind their respective
     firewalls.  At this stage, the exact parameters of the SAs (e.g.,
     algorithms to use, encapsulation mode, lifetime) and the identities of
     the communicating parties (hosts, networks, etc.) are specified.  The
     reason for the existence of Quick Mode is to allow for fast SA setup,
     once the more heavy-weight Main Mode has been completed.  Generally,
     Quick Mode uses the key material derived from Main Mode to provide keys
     to the IPsec transforms to be used.

     Alternatively, a new Diffie-Hellman computation may be performed, which
     significantly slows down the exchange, but at the same time provides
     Perfect Forward Secrecy (PFS).  Briefly, this means that even should an
     attacker manage to break long-term keys used in other sessions (or,
     specifically, if an attacker breaks the Diffie-Hellman exchange performed
     during Main Mode), they will not be able to decrypt this traffic.
     Normally, no PFS is provided (the key material used by the IPsec SAs
     established as a result of this exchange will be derived from the key
     material of the Main Mode exchange), allowing for a faster Quick Mode
     exchange (no public key computations).

     IKE proposals are "suggestions" by the initiator of an exchange to the
     responder as to what protocols and attributes should be used on a class
     of packets.  For example, a given exchange may ask for ESP with 3DES and
     MD5 and AH with SHA1 (applied successively on the same packet), or just
     ESP with Blowfish and RIPEMD-160.  The responder examines the proposals
     and determines which of them are acceptable, according to policy and any
     credentials.

     The following paragraphs assume some knowledge of the contents of the
     keynote(4) and keynote(5) man pages.

     In the KeyNote policy model for IPsec, no distinction is currently made
     based on the ordering of AH and ESP in the packet.  Should this change in
     the future, an appropriate attribute (see below) will be added.

     The goal of security policy for IKE is thus to determine, based on local
     policy (provided in the isakmpd.policy file), credentials provided during
     the IKE exchanges (or obtained through other means), the SA attributes
     proposed during the exchange, and perhaps other (side-channel)
     information, whether a pair of SAs should be installed in the system (in
     fact, whether both the IPsec SAs and the flows should be installed).  For
     each proposal suggested by or to the remote IKE daemon, the KeyNote
     system is consulted as to whether the proposal is acceptable based on
     local policy (contained in isakmpd.policy, in the form of policy
     assertions) and remote credentials (e.g., KeyNote credentials or X509
     certificates provided by the remote IKE daemon).

     isakmpd.policy is simply a flat ascii(7) file containing KeyNote policy
     assertions, separated by blank lines (note that KeyNote assertions may
     not contain blank lines).  isakmpd.policy is read when isakmpd(8) is
     first started, and every time it receives a SIGHUP signal.  The new
     policies read will be used for all new Phase 2 (IPsec) SAs established
     from that point on (even if the associated Phase 1 SA was already
     established when the new policies were loaded).  The policy change will
     not affect already established Phase 2 SAs.

     For more details on KeyNote assertion format, please see keynote(5).
     Briefly, KeyNote policy assertions used in IKE have the following
     characteristics:

     o   The Authorizer field is typically "POLICY" (but see the examples
         below, for use of policy delegation).

     o   The Licensees field can be an expression of passphrases used for
         authentication of the Main Mode exchanges, and/or public keys
         (typically, X509 certificates), and/or X509 distinguished names.

     o   The Conditions field contains an expression of attributes from the
         IPsec policy action set (see below as well as the keynote syntax man
         page for more details).

     o   The ordered return-values set for IPsec policy is "false, true".

     For an explanation of these fields and their semantics, see keynote(5).

     For example, the following policy assertion:

         Authorizer: "POLICY"
         Licensees: "passphrase:foobar" || "x509-base64:abcd```" ||
           "passphrase-md5-hex:3858f62230ac3c915f300c664312c63f" ||
           "passphrase-sha1-hex:8843d7f92416211de9ebb963ff4ce28125932878"
         Conditions: app_domain ``` "IPsec policy" && esp_present ``` "yes"
                     && esp_enc_alg != "null" -> "true";

     says that any proposal from a remote host that authenticates using the
     passphrase "foobar" or the public key contained in the X509 certificate
     encoded as "abcd```" will be accepted, as long as it contains ESP with a
     non-null algorithm (i.e., the packet will be encrypted).  The last two
     authorizers are the MD5 and SHA1 hashes respectively of the passphrase
     "foobar".  This form may be used instead of the "passphrase:..." one to
     protect the passphrase as included in the policy file (or as distributed
     in a signed credential).

     The following policy assertion:

         Authorizer: "POLICY"
         Licensees: "DN:/CN=CA Certificate"
         Conditions: app_domain ``` "IPsec policy" && esp_present ``` "yes"
                     && esp_enc_alg != "null" -> "true";

     is similar to the previous one, but instead of including a complete X509
     credential in the Licensees field, only the X509 certificate's Subject
     Canonical Name needs to be specified (note that the "DN:" prefix is
     necessary).

     KeyNote credentials have the same format as policy assertions, with one
     difference: the Authorizer field always contains a public key, and the
     assertion is signed (and thus its integrity can be cryptographically
     verified).  Credentials are used to build chains of delegation of
     authority.  They can be exchanged during an IKE exchange, or can be
     retrieved through some out-of-band mechanism (no such mechanism is
     currently supported in this implementation however).  See isakmpd.conf(5)
     on how to specify what credentials to send in an IKE exchange.

     Passphrases that appear in the Licensees field are encoded as the string
     "passphrase:", followed by the passphrase itself (case-sensitive).
     Alternatively (and preferably), they may be encoded using the
     "passphrase-md5-hex:" or "passphrase-sha1-hex:" prefixes, followed by the
     md5(1) or sha1(1) hash of the passphrase itself, encoded as a hexadecimal
     string (using lower-case letters only).

     When X509-based authentication is performed in Main Mode, any X509
     certificates received from the remote IKE daemon are converted to very
     simple KeyNote credentials.  The conversion is straightforward: the
     issuer of the X509 certificate becomes the Authorizer of the KeyNote
     credential, the subject becomes the only Licensees entry, while the
     Conditions field simply asserts that the credential is only valid for
     "IPsec policy" use (see the app_domain action attribute below).

     Similarly, any X509 CA certificates present in the directory pointed to
     by the appropriate isakmpd.conf(5) entry are converted to such pseudo-
     credentials.  This allows one to write KeyNote policies that delegate
     specific authority to CAs (and the keys those CAs certify, recursively).

     For more details on KeyNote assertion format, see keynote(5).

     Information about the proposals, the identity of the remote IKE daemon,
     the packet classes to be protected, etc. are encoded in what is called an
     action set.  The action set is composed of name-value attributes, similar
     in some ways to shell environment variables.  These values are
     initialized by isakmpd(8) before each query to the KeyNote system, and
     can be tested against in the Conditions field of assertions.  See
     keynote(4) and keynote(5) for more details on the format and semantics of
     the Conditions field.

     Note that assertions and credentials can make references to non-existent
     attributes without catastrophic failures (access may be denied, depending
     on the overall structure, but will not be accidentally granted).  One
     reason for credentials referencing non-existent attributes is that they
     were defined within a specific implementation or network only.

     In the following attribute set, IPv4 addresses are encoded as ASCII
     strings in the usual dotted-quad format.  However, all quads are three
     digits long.  For example, the IPv4 address 10.128.1.12 would be encoded
     as 010.128.001.012.  Similarly, IPv6 addresses are encoded in the
     standard x:x:x:x:x:x:x:x format, where the 'x's are the hexadecimal
     values of the eight 16-bit pieces of the address.  All 'x's are four
     digits long.  For example, the address 1080:0:12:0:8:800:200C:417A would
     be encoded as 1080:0000:0012:0000:0008:0800:200C:417A.

     The following attributes are currently defined:

     ah_auth_alg
             One of ________, ________, _______, ____, _____________,
             _____________, _____________, or ___________.  based on the
             authentication method specified in the AH proposal.

     ah_ecn, esp_ecn, comp_ecn
             Set to ___ or __, based on whether ECN was requested for the
             IPsec tunnel.

     ah_encapsulation, esp_encapsulation, comp_encapsulation
             Set to ______ or _________, based on the AH, ESP, and compression
             proposal.

     ah_group_desc, esp_group_desc, comp_group_desc
             The Diffie-Hellman group identifier from the AH, ESP, and
             compression proposal, used for PFS during Quick Mode (see the pfs
             attribute above).  If more than one of these attributes are set
             to a value other than zero, they should have the same value (in
             valid IKE proposals).  Valid values are 1 (768-bit MODP), 2
             (1024-bit MODP), 3 (155-bit EC), 4 (185-bit EC), and 5 (1536-bit
             MODP).

     ah_hash_alg
             One of ___, ___, ______, ________, ________, ________, or ___,
             based on the hash algorithm specified in the AH proposal.  This
             attribute describes the generic transform to be used in the AH
             authentication.

     ah_key_length, esp_key_length
             The number of key bits to be used by the authentication and
             encryption algorithms respectively (for variable key-size
             algorithms).

     ah_key_rounds, esp_key length
             The number of rounds of the authentication and encryption
             algorithms respectively (for variable round algorithms).

     ah_life_kbytes, esp_life_kbytes, comp_life_kbytes
             Set to the lifetime of the AH, ESP, and compression proposal, in
             kbytes of traffic.  If no lifetime was proposed for the
             corresponding protocol (e.g., there was no proposal for AH), the
             corresponding attribute will be set to zero.

     ah_life_seconds, esp_life_seconds, comp_life_seconds
             Set to the lifetime of the AH, ESP, and compression proposal, in
             seconds.  If no lifetime was proposed for the corresponding
             protocol (e.g., there was no proposal for AH), the corresponding
             attribute will be set to zero.

     ah_present, esp_present, comp_present
             Set to ___ if an AH, ESP, or compression proposal was received
             respectively, __ otherwise.

     app_domain
             Always set to _____ ______.

     comp_alg
             One of ___, _______, ___, or ______, based on the compression
             algorithm specified in the compression proposal.

     comp_dict_size
             Specifies the log2 maximum size of the dictionary, according to
             the compression proposal.

     comp_private_alg
             Set to an integer specifying the private algorithm in use,
             according to the compression proposal.

     doi     Always set to _____.

     esp_auth_alg
             One of ________, ________, _______, ____, _____________,
             _____________, _____________, or ___________ based on the
             authentication method specified in the ESP proposal.

     esp_enc_alg
             One of ___, ________, ____, ___, ____, ____, ________, _____,
             ________, ___, ____, or ___, based on the encryption algorithm
             specified in the ESP proposal.

     GMTTimeOfDay
             Set to the UTC date/time, in YYYYMMDDHHmmSS format.

     initiator
             Set to ___ if the local daemon is initiating the Phase 2 SA, __
             otherwise.

     local_negotiation_address
             Set to the IPv4 or IPv6 address of the local interface used by
             the local IKE daemon for this exchange.

     LocalTimeOfDay
             Set to the local date/time, in YYYYMMDDHHmmSS format.

     pfs     Set to ___ if a Diffie-Hellman exchange will be performed during
             this Quick Mode, __ otherwise.

     phase_1
             Set to __________ if aggressive mode was used to establish the
             Phase 1 SA, or ____ if main mode was used instead.

     phase1_group_desc
             The Diffie-Hellman group identifier used in IKE Phase 1.  Takes
             the same values as _____________.

     remote_filter, local_filter, remote_id
             When the corresponding filter_type specifies an address range or
             subnet, these are set to the upper and lower part of the address
             space separated by a dash ('-') character (if the type specifies
             a single address, they are set to that address).

             For FQDN and User FQDN types, these are set to the respective
             string.  For Key ID, these are set to the hexadecimal
             representation of the associated byte string (lower-case letters
             used) if the Key ID payload contains non-printable characters.
             Otherwise, they are set to the respective string.

             For ASN1 DN, these are set to the text encoding of the
             Distinguished Name in the payload sent or received.  The format
             is the same as that used in the Licensees field.

     remote_filter_addr_lower, local_filter_addr_lower, remote_id_addr_lower
             When the corresponding filter_type is ____ _______ or ____
             _______, these contain the respective address.  For ____ _____ or
             ____ _____, these contain the lower end of the address range.
             For ____ ______ or ____ ______, these contain the lowest address
             in the specified subnet.

     remote_filter_addr_upper, local_filter_addr_upper, remote_id_addr_upper
             When the corresponding filter_type is ____ _______ or ____
             _______, these contain the respective address.  For ____ _____ or
             ____ _____, they contain the upper end of the address range.  For
             ____ ______ or ____ ______, they contain the highest address in
             the specified subnet.

     remote_filter_port, local_filter_port, remote_id_port
             Set to the transport protocol port.

     remote_filter_proto, local_filter_proto, remote_id_proto
             Set to _______, ___, ___, or the transport protocol number,
             depending on the transport protocol set in the IDci, IDcr, and
             Main Mode peer ID respectively.

     remote_filter_type, local_filter_type, remote_id_type
             Set to ____ _______, ____ _____, ____ ______, ____ _______, ____
             _____, ____ ______, ____, ____ ____, ____ __, ____ __, or ___ __,
             based on the Quick Mode Initiator ID, Quick Mode Responder ID,
             and Main Mode peer ID respectively.

     remote_negotiation_address
             Set to the IPv4 or IPv6 address of the remote IKE daemon.

FILES
     ___________________________  The default isakmpd(8) policy configuration
                                  file.

EXAMPLES
         Authorizer: "POLICY"
         Comment: This bare-bones assertion accepts everything



         Authorizer: "POLICY"
         Licensees: "passphrase-md5-hex:10838982612aff543e2e62a67c786550"
         Comment: This policy accepts anyone using shared-secret
                  authentication using the password mekmitasdigoat,
                  and does ESP with some form of encryption (not null).
         Conditions: app_domain ``` "IPsec policy" &&
                     esp_present ``` "yes" &&
                     esp_enc_alg != "null" -> "true";



         Authorizer: "POLICY"
         Licensees: "subpolicy1" || "subpolicy2"
         Comment: Delegate to two other sub-policies, so we
                  can manage our policy better. Since these subpolicies
                  are not "owned" by a key (and are thus unsigned), they
                  have to be in isakmpd.policy.
         Conditions: app_domain ``` "IPsec policy";



         KeyNote-Version: 2
         Licensees: "passphrase-md5-hex:9c42a1346e333a770904b2a2b37fa7d3"
         Conditions: esp_present ``` "yes" -> "true";
         Authorizer: "subpolicy1"



         Conditions: ah_present ``` "yes" ->
                        {
                            ah_auth_alg ``` "md5" -> "true";
                            ah_auth_alg ``` "sha" &&
                            esp_present ``` "no" -> "true";
                        };
         Licensees: "passphrase:otherpassword" ||
            "passphrase-sha1-hex:f5ed6e4abd30c36a89409b5da7ecb542c9fbf00f"
         Authorizer: "subpolicy2"



         keynote-version: 2
         comment: this is an example of a policy delegating to a CN.
         authorizer: "POLICY"
         licensees: "DN:/CN=CA Certificate/emailAddress=ca@foo.bar.com"



         keynote-version: 2
         comment: This is an example of a policy delegating to a key.
         authorizer: "POLICY"
         licensees: "x509-base64:MIICGDCCAYGgAwIBAgIBADANBgkqhkiG9w0BAQQ\
                     FADBSMQswCQYDVQQGEwJHQjEOMAwGA1UEChMFQmVuQ28xETAPBg\
                     NVBAMTCEJlbkNvIENBMSAwHgYJKoZIhvcNAQkBFhFiZW5AYWxnc\
                     m91cC5jby51azAeFw05OTEwMTEyMjQ5MzhaFw05OTExMTAyMjQ5\
                     MzhaMFIxCzAJBgNVBAYTAkdCMQ4wDAYDVQQKEwVCZW5DbzERMA8\
                     GA1UEAxMIQmVuQ28gQ0ExIDAeBgkqhkiG9w0BCQEWEWJlbkBhbG\
                     dyb3VwLmNvLnVrMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBg\
                     QCxyAte2HEVouXg1Yu+vDihbnjDRn+6k00Rv6cZqbwA3BQ30mC/\
                     3TFJ09VGXCaM0UKfpnxIpkBYLmOA3FWkKI0RvPU7E1AhKkhC1Ds\
                     PSBFjYHrB15T5lYzgfwKJCIxTDzZDx2iobUgPa0FRNGVUjpQ4/k\
                     MJ2BF4Wh7zY3X08rMzsQIDAQABMA0GCSqGSIb3DQEBBAUAA4GBA\
                     DWJ5pbTcE7iKHWLQTMYiz8i9jGi5+Eo1yr1Bab90tgaGQV0zrRH\
                     jDHgAAy1h8WSXuyQrXfgbx2rnWFPhx9CfmuAXn7sZmQE3mnUqeP\
                     ZL2dW87jdBGqtoUdNcoz5zKBkC943yasNui/O01MiqgadTThTJH\
                     d1Pn17LbJC1ZVRNjR5"
         conditions: app_domain ``` "IPsec policy" && doi ``` "ipsec" &&
                 pfs ``` "yes" && esp_present ``` "yes" && ah_present ``` "no" &&
                 (esp_enc_alg ``` "3des" || esp_enc_alg ``` "aes") -> "true";



         keynote-version: 2
         comment: This is an example of a credential, the signature does
                  not really verify (although the keys are real).
         licensees: "x509-base64:MIICGDCCAYGgAwIBAgIBADANBgkqhkiG9w0BAQQ\
                     FADBSMQswCQYDVQQGEwJHQjEOMAwGA1UEChMFQmVuQ28xETAPBg\
                     NVBAMTCEJlbkNvIENBMSAwHgYJKoZIhvcNAQkBFhFiZW5AYWxnc\
                     m91cC5jby51azAeFw05OTEwMTEyMzA2MjJaFw05OTExMTAyMzA2\
                     MjJaMFIxCzAJBgNVBAYTAkdCMQ4wDAYDVQQKEwVCZW5DbzERMA8\
                     GA1UEAxMIQmVuQ28gQ0ExIDAeBgkqhkiG9w0BCQEWEWJlbkBhbG\
                     dyb3VwLmNvLnVrMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBg\
                     QDaCs+JAB6YRKAVkoi1NkOpE1V3syApjBj0Ahjq5HqYAACo1JhM\
                     +QsPwuSWCNhBT51HX6G6UzfY3mOUz/vou6MJ/wor8EdeTX4nucx\
                     NSz/r6XI262aXezAp+GdBviuJZx3Q67ON/IWYrB4QtvihI4bMn5\
                     E55nF6TKtUMJTdATvs/wIDAQABMA0GCSqGSIb3DQEBBAUAA4GBA\
                     MaQOSkaiR8id0h6Zo0VSB4HpBnjpWqz1jNG8N4RPN0W8muRA2b9\
                     85GNP1bkC3fK1ZPpFTB0A76lLn11CfhAf/gV1iz3ELlUHo5J8nx\
                     Pu6XfsGJm3HsXJOuvOog8Aean4ODo4KInuAsnbLzpGl0d+Jqa5u\
                     TZUxsyg4QOBwYEU92H"
         authorizer: "x509-base64:MIICGDCCAYGgAwIBAgIBADANBgkqhkiG9w0BAQQ\
                      FADBSMQswCQYDVQQGEwJHQjEOMAwGA1UEChMFQmVuQ28xETAPBg\
                      NVBAMTCEJlbkNvIENBMSAwHgYJKoZIhvcNAQkBFhFiZW5AYWxnc\
                      m91cC5jby51azAeFw05OTEwMTEyMjQ5MzhaFw05OTExMTAyMjQ5\
                      MzhaMFIxCzAJBgNVBAYTAkdCMQ4wDAYDVQQKEwVCZW5DbzERMA8\
                      GA1UEAxMIQmVuQ28gQ0ExIDAeBgkqhkiG9w0BCQEWEWJlbkBhbG\
                      dyb3VwLmNvLnVrMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBg\
                      QCxyAte2HEVouXg1Yu+vDihbnjDRn+6k00Rv6cZqbwA3BQ30mC/\
                      3TFJ09VGXCaM0UKfpnxIpkBYLmOA3FWkKI0RvPU7E1AhKkhC1Ds\
                      PSBFjYHrB15T5lYzgfwKJCIxTDzZDx2iobUgPa0FRNGVUjpQ4/k\
                      MJ2BF4Wh7zY3X08rMzsQIDAQABMA0GCSqGSIb3DQEBBAUAA4GBA\
                      DWJ5pbTcE7iKHWLQTMYiz8i9jGi5+Eo1yr1Bab90tgaGQV0zrRH\
                      jDHgAAy1h8WSXuyQrXfgbx2rnWFPhx9CfmuAXn7sZmQE3mnUqeP\
                      ZL2dW87jdBGqtoUdNcoz5zKBkC943yasNui/O01MiqgadTThTJH\
                      d1Pn17LbJC1ZVRNjR5"
     conditions: app_domain ``` "IPsec policy" && doi ``` "ipsec" &&
                 pfs ``` "yes" && esp_present ``` "yes" && ah_present ``` "no" &&
                 (esp_enc_alg ``` "3des" || esp_enc_alg ``` "aes") -> "true";
     Signature: "sig-x509-sha1-base64:ql+vrUxv14DcBOQHR2jsbXayq6T\
                 mmtMiUB745a8rjwSrQwh+KIVDlUrghPnqhSIkWSDi9oWWMbfg\
                 mkdudZ0wjgeTLMI2NI4GibMMsToakOKMex/0q4cpdpln3DKcQ\
                 IcjzRv4khDws69FT3QfELjcpShvbLrXmh1Z00OFmxjyqDw="

SEE ALSO
     ipsec(4), keynote(4), keynote(5), isakmpd(8)

BUGS
     A more sane way of expressing IPv6 address ranges is needed.

OpenBSD 4.8                      June 7, 2010                      OpenBS
