# Automation Scripts (SoapUI)
code (groovy script)

#scripts to retrieve data from the api response and check for the validation

**(raw data to json )**
import groovy.json.JsonSlurper
def response = messageExchange.response.responseContent
assert response != ""
def jsonslurper = new JsonSlurper().parseText(response)

**(for setting the random keyword/tenant/user)**
def id1 = UUID.randomUUID().toString()
String[] keyword;
keyword = id1.split('-');
context.testCase.setPropertyValue("random_no",keyword[4])
//log.info keyword[4]

**(for storing the value at the project level)**
import groovy.json.JsonSlurper
def response = messageExchange.response.responseContent
assert response != ""
def jsonslurper = new JsonSlurper().parseText(response)
assert jsonslurper.token.access_token != ""
context.testCase.testSuite.project.setPropertyValue("x_auth_token", jsonslurper.token.access_token)

**(for comapring any two strings)**
import groovy.json.JsonSlurper
def response = messageExchange.response.responseContent
assert response != ""
def jsonslurper = new JsonSlurper().parseText(response)
assert jsonslurper.status != "" && jsonslurper.status != null
assert jsonslurper.message != "" && jsonslurper.message != null
def my_msg = "Invalid credentials. Please check the provided auth_values".toUpperCase()
//log.info msg
def response_msg = jsonslurper.message.toUpperCase()
//log.info response_msg
assert my_msg == response_msg

**(for importing the roles for the account onboarding using groovy script )**
import groovy.json.JsonSlurper
import groovy.json.*
def json_1 = testRunner.testCase.getTestStepByName("List all roles in a tenant").getPropertyValue("response")
def response1 = new JsonSlurper().parseText(json_1)
ArrayList<String> roles = new ArrayList<String>()
if(response1.roles != [])
{
    for(int i = 0 ; i < response1.roles.size() ; i ++)
    {
        roles.add(response1.roles[i].id)
    }
}
else
{
    testRunner.fail "No roles data is fetched."
}
return roles
def requestBody_for_configure_snow_account_and_into_integrated_tools_Step = """
{
    "settings_type": "none",
    "roles": ${JsonOutput.toJson(roles)},
    "settings": {
        "product_activation": [
            {
                "resource_category": "Request_Management",
                "resource": []
            },
            {
                "resource_category": "Incident_Management",
                "resource": []
            },
            {
                "resource_category": "Change_Management",
                "resource": []
            },
            {
                "resource_category": "Configuration_Management",
                "resource": [
                    "CMDB"
                ]
            }
        ],
        "tools_configuration": {}
    }
}
"""
//log.info requestBody_for_configure_snow_account_and_into_integrated_tools_Step
context.testCase.testSteps["Configure snow account and add into integrated tools"].setPropertyValue("request", requestBody_for_configure_snow_account_and_into_integrated_tools_Step)

**(loop to search for a particular cloud account)**
if(jsonslurper.cloud_accounts != {} || [])
{
	for(def i = 0 ; i < jsonslurper.cloud_accounts.size() ; i ++)
	{
		if(jsonslurper.cloud_accounts[i].cloud_account_name == "CS_SaaS_QA_INFRA_120723-expiry")
		{
			context.testCase.testSuite.setPropertyValue("account_id_Azure", jsonslurper.cloud_accounts[i].cloud_account_id)
			log.info jsonslurper.cloud_accounts[i].cloud_account_id
			break
		}
	}
}

def vmtemplate = jsonslurper.data.templates.find {
	template -> template.name == "AWS_Provision_VirtualMachine_Ubuntu1"
}

def stoptemplate = jsonslurper.data.templates.find {
	template -> template.name == "AWS_Provision_VirtualMachine_Ubuntu1"
}
assert vmtemplate != null
assert stoptemplate != null

context.testCase.testSuite.project.setPropertyValue("AWS_blueprint_template_name", vmtemplate.name)
context.testCase.testSuite.project.setPropertyValue("AWS_blueprint_template_id", vmtemplate.id)


**(for checking and returning the message)**
import groovy.json.JsonSlurper
def json_1 = testRunner.testCase.getTestStepByName("Activity_Page").getPropertyValue("response")
def jsonslurper = new JsonSlurper().parseText(json_1)

def flag = false
if(jsonslurper.data.cloud_account_configuration != [])
{   
	for(def i = 0 ; i < jsonslurper.data.cloud_account_configuration.size() ; i ++)
	{
		if(jsonslurper.data.cloud_account_configuration[i].cloud_account_name == "AWS_547045142213_ssm_testing")
		{		
			flag = true
			return "Template has been applied"
		}
		
	}
}
else
{
	testRunner.fail "No data fetched for cloud accounts."
}
if(flag == false)
{
	testRunner.fail "Template could not be applied"
}

**(for getting the value from the suite level and storing it in the new variable)**
import groovy.json.JsonSlurper
def json_1 = testRunner.testCase.getTestStepByName("Activity_Page").getPropertyValue("response")
def jsonslurper = new JsonSlurper().parseText(json_1)

def flag = false
def account_id_GCP = context.testCase.testSuite.getPropertyValue("account_id_GCP")
if(jsonslurper.data.cloud_account_configuration != {} || [])
{   
		if(jsonslurper.data.cloud_account_configuration[0].cloud_account_id == account_id_GCP)
		{		
			flag = true
			return "Template has been applied"
		}
		
  }
else
{
	testRunner.fail "No data fetched for cloud accounts."
}

**(storing a value from the nested JSON when 1 json is in array)**
import groovy.json.JsonSlurper
def response = messageExchange.response.responseContent
assert response != ""
def jsonslurper = new JsonSlurper().parseText(response)


def oci_compartment_name
def oci_compartment_id
//jsonslurper.data.service_accounts[0].metadata.compartments.find {
//	compartments -> compartments.description == "Corestack Integration"
//}

for(def compartments in jsonslurper.data.service_accounts[0].metadata.compartments) {
	if(compartments.description == "Corestack Integration") {
		oci_compartment_id = compartments.compartment_id
		oci_compartment_name = compartments.name
		break
	}
}



 log.info oci_compartment_name
 log.info oci_compartment_id


 context.testCase.testSuite.project.setPropertyValue("oci_compartment_name", oci_compartment_name)
 context.testCase.testSuite.project.setPropertyValue("oci_compartment_id", oci_compartment_id)

**(script to trigger an API for 25 mins and comparing the parameters/status)**
import groovy.json.JsonSlurper

def cloudAccountId = context.testCase.testSuite.project.getPropertyValue("cloud_account_id_AWS")
def monitoringTemplateName = context.testCase.testSuite.project.getPropertyValue("AWS-monitoring-template_name")
//log.info monitoringTemplateName

def listCloudAccountconfigApi  = context.testCase.testSteps["List Cloud Account Configuration"]
log.info listCloudAccountconfigApi
def startTime = System.currentTimeMillis()
while( System.currentTimeMillis() - startTime <= 1500000) {
	listCloudAccountconfigApi.run(testRunner, context)
	
	def rawResponse = testRunner.testCase.getTestStepByName("List Cloud Account Configuration").getPropertyValue("response")
	def response = new JsonSlurper().parseText(rawResponse)
	
	for(def cloudAccountDetails in response.cloud_account_configuration) {
		if(cloudAccountDetails.cloud_account_id == cloudAccountId && cloudAccountDetails.monitoring_template == monitoringTemplateName) {
			cloudAccountJson = cloudAccountDetails
			if(cloudAccountJson.status == "partially_completed" || cloudAccountJson.status == "completed") {
				return "Template applied on cloud account."
			}
	
			if(cloudAccountJson.status == "failed")  {
				testRunner.fail("Template application is Failed on cloud account..\nTemplate Application status : " + cloudAccountJson.status)
				return
			}
	
			if(cloudAccountJson.status == "skipped")  {
				testRunner.fail("Template application is Skipped on cloud account..\nTemplate Application status : " + cloudAccountJson.status)
				return
			}
		}
	}
	sleep(180000)
}

testRunner.fail("Template status is still not Partially_Completed / completed..")
