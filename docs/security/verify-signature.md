---
title: Verify signature
sidebar_position: 3
---

In Light Saml the following SAML documents: AuthnRequest, Response, LogoutRequest, and LogoutResponse, have been generalized
into abstract ``SamlMessage`` class. It holds attributes and elements common to those documents, like ID, IssueInstance, Issuer...
Also, it has the signature property. For the serialization purposes it has to be a SignatureWriter, while after deserialization
it is a SignatureReader instance.

Signature can be validated with ``SignatureReader::validate()`` method passing the public key argument. It will throw exception
if signature validation fails, or return true if it succeeds.

Following example shows how you can validate the signature of a SAML AuthnRequest.

```php
<?php
$xml = '<AuthnRequest><ds:Signature>...</ds:Signature>...</AuthnRequest>';

$deserializationContext = new \LightSaml\Model\Context\DeserializationContext();
$deserializationContext->getDocument()->loadXML($xml);
$authnRequest = new \LightSaml\Model\Protocol\AuthnRequest();
$authnRequest->deserialize($deserializationContext->getDocument()->firstChild, $deserializationContext);

$key = \LightSaml\Credential\KeyHelper::createPublicKey(
    \LightSaml\Credential\X509Certificate::fromFile('idp.crt')
);

/** @var \LightSaml\Model\XmlDSig\SignatureXmlReader $signatureReader */
$signatureReader = $authnRequest->getSignature();

try {
    $ok = $signatureReader->validate($key);

    if ($ok) {
        print "Signature OK\n";
    } else {
        print "Signature not validated";
    }
} catch (\Exception $ex) {
    print "Signature validation failed\n";
}
```

Similarly, you could verify the signature of other ``SamlMessage`` descendants, like ``Response``, simply by instantiating it
instead of the AuthnRequest from above. The rest of the code will remain the same.
