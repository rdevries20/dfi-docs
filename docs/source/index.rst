DFI blacklisting system
=======================

This piece of documentation will teach you:

- How to setup the blacklisting system
- How to use the blacklisting system
- How to blackling system works (including what all classes do)
- How to add new rule logic


**A short description**:

The blacklisting system read binaries from a folder, submits these to a `Cuckoo Sandbox <https://cuckoosandbox.org/>`_ instance for analysis,
downloads the results, reads these reports and applies its own logic to filter out hashes of malicious
binaries.

The md5 hashes of the malicious binaries are stored and retrievable through a web API.
You could use these hashes for the creation of IDS alerts.

Contents:

.. toctree::
   howitworks/index