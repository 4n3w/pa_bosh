Hosted here -> http://bosh-docs.cfapps.io/

To update -> 
```bash
mkdocs build
cd site
cf push -b staticfile_buildpack bosh-docs
```
 
