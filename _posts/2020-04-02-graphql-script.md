---
layout: post
title: Speed up compile time with script dependency
subtitle: Large projects have lots of scripts and take few minutes or even dozen of minutes to build.  
date:   2020-04-02
background: https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fbash-script.jpg?alt=media&token=04b47378-e3d5-4343-885d-7c46e9cb32a7
---
Large projects have lots of scripts and take few minutes or even dozens of minutes to build. Mine takes 4-5 minutes for every compile time. It's really inconvenience when implement UI, change spacing or colors. I have to change that.

# Online Instructions
I followed some instructions to speed it up. You can find it with ease. 

# Reduce not important scripts
Open your `Build Phrase` and check your scripts. I had 4 running scripts, generating GraphQL API, 2 Pods related scripts, Sentry scripts (upload .dSYM). 

I notice generating GraphQL API and Sentry scripts here. Why do you need it run everytime we compile? 

### Get rid of GraphQL script

> **How often do we change GraphQL schemas?**
> Only when implement new features.

This is where I started. 
Remove GraphQL script and add it into a bash script file. Run script when you need to recompile schema. Here is my script: 
```
# (1)
apollo schema:download --endpoint=server_endpoint path_to_schema_file/schema.json
# (2)
SCHEMA_PATH=$PWD"/project_name"
# (3)
$PWD/Pods/Apollo/scripts/run-bundled-codegen.sh codegen:generate --target=swift --includes='./**/*.graphql' --localSchemaFile="$SCHEMA_PATH/Services/GraphQL/schema.json" $SCHEMA_PATH/Services/GraphQL/API.swift
```
(1): Download latest schema from your server. `server_endpoint` is your server path. Usually it's like this `https://api.company_name.com/graphql`

(2): $PWD: get your current directory and point it to your project

(3): Now generate file. Please make sure you change `$SCHEMA_PATH/Services/GraphQL/schema.json` and `$SCHEMA_PATH/Services/GraphQL/API.swift` to yours. 

**Notes:**
You have to run `chmod u+r+x downloadSchema.sh` at the first time run the bash file.
Then run, `./downloadSchema.sh`. 

Later, just run download script. 
That's it. I save minutes with this. 


### Remove uploading script
> **How often do we need to upload .dSYM?**
> Only when release new version

Ask yourself, do you really want to do this? Some extra work is needed when release to upload .dSYM to Sentry (crashed report service I used). But for me, it's worth to do. 

Some extra work: 
- Release 
- Download dSYM from AppstoreConnect when finish processing
- Upload manually

If it doesn't matter with you, please try this script. 
`sentry-cli upload-dif --org company_name --project project_name --log-level=debug appDsyms.zip`

I added it into my `.zshrc`as an alias, so after download, just run alias and done. 

# Conclusion
This is my experience, maybe it doesn't work in your project. Please feel free to discuss [here](https://github.com/nguyentruongky/nguyentruongky.github.io/issues/2).

> Enjoy coding. 




