

oc policy add-role-to-user system:image-puller system:serviceaccount:petclinic-dev:petclinic -n cicd
oc policy add-role-to-user system:image-puller system:serviceaccount:petclinic-uat:petclinic -n cicd

oc adm policy add-role-to-user system:image-pullers system:serviceaccount:petclinic-dev:petclinic -n cicd

oc adm policy add-role-to-user system:image-pullers system:serviceaccount:petclinic-uat:petclinic -n cicd