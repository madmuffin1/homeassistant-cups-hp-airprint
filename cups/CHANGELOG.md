# Changelog

## 1.9-beta3

- Start cups-browsed so not only discovery but actual printing works
- Known issues: In one test setup this caused the client to receive a "printing failed" error after the document was successfully printed. The job needs to be canceled on the client or will be re-queued. 

