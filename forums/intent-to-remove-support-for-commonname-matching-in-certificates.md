# Intent to Remove: Support for commonName matching in certificates

https://groups.google.com/a/chromium.org/forum/#!msg/security-dev/IGT2fLJrAeo/csf_1Rh1AwAJ

Author: Ryan Sleevi

Date: Jan 28, 2017

## Summary

Certificates have two ways to express the domain/IP they're bound to - one which is unstructured and ambiguous (commonName), and one which is well-defined (subjectAltName). In the absence of any subjectAltNames, Chrome currently falls back to comparing the domain against the commonName, if present.

This proposal is to remove that fallback path; in effect, requiring a subjectAltName. Ideally, we would do this for all certificates (publicly trusted and privately trusted), but if there are concerns about compat risk, we can restrict it to publicly trusted certificates.

## Motivation

Since 1997 (X.509v3's ratification), certificates have had two ways to express a binding to a domain name - either via the commonName attribute within the certificate's subject, or via the explicitly typed (dNSName or iPAddress) of the SubjectAlternativeName Extension.

> **ratification**: the ratification of a treaty or written agreement is the process of ratifying it. 

Since RFC 2818 (published in 2000, first drafted in January 1998), the use of the commonName field has been considered deprecated, because it's ambiguous and untyped - that is, it might contain a human-readable name or it might be a domain name.

The use of the subjectAlternativeName fields leaves it unambiguous whether a certificate is expressing a binding to an IP address or a domain name, and is fully defined in terms of its interaction with Name Constraints. commonName, however, is ambiguous, and because of this, support for the commonName has been a source of security bugs - in both Chrome and the libraries it uses and within the TLS ecosystem at large.

## Compatibility Risk

Low. RFC 2818 has deprecated this for nearly two decades, and the Baseline Requirements (which all publicly trusted CAs must abide by) has required the presence of a subjectAltName since 2012.

Mozilla Firefox already requires the subjectAltName for any newly issued publicly trusted certificates since Firefox 48 ( https://bugzilla.mozilla.org/show_bug.cgi?id=1245280 ).

## Usage information

From a metrics perspective, less than .002% of publicly trusted certificate validations would require this behaviour (Net.CertCommonNameFallback minus Net.CertCommonNameFallbackPrivateCA).

As 1.57% of privately-trusted CA certificates rely on this behaviour (or 0.1% of all certificate validations), it may be premature to deprecate it for privately-trusted CAs; alternatively, we could remove it with an enterprise policy to allow it for a limited number of releases. Unfortunately, despite being deprecated for nearly 20 years, it's unlikely we'd be able to drive this number down (and improve the security of the ecosystem) without taking further action.