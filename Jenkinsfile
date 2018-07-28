pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        git(url: 'http://164.99.103.45/gitlab/Javeed/filr_rest.git', branch: 'master', poll: true)
        sh '''cd /root/BBS/home:Ajaveed:branches:Filr:3.0.0:Update/filr-ssf-rest-lib

if [[ -d "filr-ssf-rest-lib" ]]
then
rm -rf ./filr-ssf-rest-lib
fi

git clone http://164.99.103.45/gitlab/Javeed/filr_rest.git

mv ./filr_rest ./filr-ssf-rest-lib

tar -cvjSf filr-ssf-rest-lib.tar.bz2 ./filr-ssf-rest-lib

osc commit -m "changed"'''
        sh '''cd /root/BBS/home:Ajaveed:branches:Filr:3.0.0:Update/filr-ssf-rest-lib
#osc rebuildpac
osc results -w
#osc buildlog OES_2018_SP1 x86_64
cd ..
osc r --xml > buildTest.xml
repo_str=$(xmllint --xpath "//resultlist/result/@repository" buildTest.xml)
repo=()
for rep in $repo_str
do
rep=${rep#repository=}
rep=${rep#\\"}
rep=${rep%\\"}
repo+=($rep)
done

#echo ${repo[0]}

arch_str=$(xmllint --xpath "//resultlist/result/@arch" buildTest.xml)
arch=()
for arc in $arch_str
do
arc=${arc#arch=}
arc=${arc#\\"}
arc=${arc%\\"}
arch+=($arc)
done

#echo ${arch[0]}

package_str=$(xmllint --xpath "//resultlist/result/status/@package" buildTest.xml)
package=()
for pac in $package_str
do
package+=($pac)
done

#echo ${package[0]}

status_str=$(xmllint --xpath "//resultlist/result/status/@code" buildTest.xml)
status_=()
for stat in $status_str
do
status_+=($stat)
done

#echo ${status[0]}

index=0;
for stat in "${status_[@]}"
do

if [ $stat == \'code="failed"\' ]
then
echo "failed"
exit 1
fi
index=$((index+1))
done

if [ -d "./binaries" ]
then
rm -r ./binaries
fi

index=0;
for rep in "${repo[@]}"
do
osc getbinaries $rep ${arch[index]}
index=$((index+1))
done'''
      }
    }
    stage('Deploy') {
      steps {
        ansiblePlaybook '/root/BBS/home:Ajaveed:branches:Filr:3.0.0:Update/deploy.yml'
      }
    }
    stage('Test') {
      steps {
        sh '''cd /root/ 

ant -f filr-qa/trunk/build.xml  clean compile restapis



'''
        junit '/root/filr-qa/trunk/reports/'
      }
    }
  }
}