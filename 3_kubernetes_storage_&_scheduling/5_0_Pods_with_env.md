# Why did we need Configuration as Data ?

* Abstraction
* Container Images are Immutable - you don't want to share personal info, because image can be public
* Serivce Discovery
* Sensitive Information

## Configuring Applications in Pods
They are different technics:
* Command Line Arguments
* [Environment Variables](5_1_Env_variables.md)
* [Secrets](5_2_Secrets.md)
* [ConfigMaps](5_3_ConfigMaps.md)