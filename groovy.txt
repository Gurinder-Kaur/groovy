job("job_6.1"){
        description("copy data to local machine (then pvc)")
        scm {
                 github('Gurinder-Kaur/jenkins' , 'master')
             }
        triggers {
                scm("* * * * *")
        }
        steps {
        shell('''sudo cp -rvf  * root/task6''')
      }
}

job("job_6.2"){
        description("check program type and run webserver on top of k8s")
        triggers {
            upstream('job_6.1','SUCCESS')
        }
        steps {
        shell('''if sudo ls /root/task6 | grep html
                    then
	if sudo kubectl get all | grep httpd-deployment
	then
	podname = $(sudo kubectl pod --selector type=html -o jsonpath="{items[0].metadata,name}")
	sudo kubectl cp /root/task6/*.html $podname:/usr/local/apache2/htdocs
	else
	sudo kubectl create -f /root/ht.yml
	sleep 50
	podname = $(sudo kubectl pod --selector type=html -o jsonpath="{items[0].metadata,name}")
	sudo kubectl cp /root/task6/*.html $podname:/usr/local/apache2/htdocs
	fi
	else
	echo "other than html not supported"
	fi 
      }
}


job("job_6.3") {
  description ("Testing")
  
  triggers {
    upstream('job_6.2', 'SUCCESS')
  }
  steps {
        shell('''statuscode = $(curl -o /dev/null -s -w "%{http_code}" http://192.168.99.104:30030/index.html) 
        if [[$statuscode == 200]]
        then
        exit 0
        else
        exit 1
        fi
        ''' )
  }
  publishers {
    extendedEmail {
      contentType('text/html')
      triggers {
        success{
          attachBuildLog(true)
          subject('Build successfull')
          content('working!!')
          recipientList('gurinder.sehra@gmail.com')
        }
        failure{
          attachBuildLog(true)
          subject('Failed build')
          content('Error')
          recipientList('gurinder.sehra@gmail.com')
        }
      }
    }
  }
}
