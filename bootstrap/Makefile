SHELL=/bin/bash
THIS_DIR=$(shell dirname $(realpath $(lastword $(MAKEFILE_LIST))))
TENANT=ccorg
HOST=api.cortex-dev.insights.ai
#HOST=localhost:8000
URL=https://${HOST}
ENVID=5a1f0a1b7c2e240001142805
KEY=cc-data/json1234
# accounts/admin/authenticate
TMP:=$(shell curl -s -X POST ${URL}/v2/admin/${TENANT}/users/authenticate -H 'content-type: application/json' -d '{"username": "ccuser1", "password": "secret"}')
AUTH_OUT='$(TMP)'
JWT=$(shell echo $(AUTH_OUT) | jq .jwt | tr -d '"')

print-%  : ; @echo $* = $($*)
	
# admin/{tenantId}/users/register
tenant.register:
	curl -s -X POST ${URL}/v2/admin/tenants/register -H 'content-type: application/json' -d '{"website":"website", "name": "Client Complaints Org", "tenantId": "${TENANT}", "email": "admin@cc.org", "first":"firstname", "last":"lastname", "phone":"111-111-1111", "address":"this street", "admin":"ccadmin", "password":"secret"}' | jq .
	sleep 10
	
user.register:
	curl -s -X POST ${URL}/v2/admin/${TENANT}/users/register -H 'content-type: application/json' -d '{"email": "ccuser1@cc.org", "first": "first", "last": "last", "password": "secret"}' | jq .
	sleep 10

catalog.saveTypes:
	curl -s -X POST ${URL}/v2/catalog/types -H "authorization: Bearer ${JWT}" -H 'content-type: text/yaml' --data-binary "@${THIS_DIR}/../demos/ClientComplaints/yaml/types.yml" | jq .

catalog.getTypes:
	curl -s -X GET ${URL}/v2/catalog/types -H "authorization: Bearer ${JWT}" -H 'content-type: text/yaml' | jq .

catalog.createAgent:
	# curl -s -X POST ${URL}/v2/catalog/agents -H "authorization: Bearer ${JWT}" -H 'content-type: text/yaml' --data-binary "@${THIS_DIR}/../../demos/Trading-Assistant/yaml/Po1BuilderAgent.yml" | jq .
	curl -s -X POST ${URL}/v2/catalog/agents -H "authorization: Bearer ${JWT}" -H 'content-type: text/yaml' --data-binary "@${THIS_DIR}/../demos/ClientComplaints/yaml/ClientComplaintsAgent.yml" | jq .

catalog.getAgent:
	curl -s -X GET ${URL}/v2/catalog/agents/default/client-complaints-agent -H "authorization: Bearer ${JWT}" -H 'content-type: application/json' | jq .
catalog.saveProcessor:
	 # curl -s -X POST ${URL}/v2/catalog/processors -H "authorization: Bearer ${JWT}" -H 'content-type: text/yaml' --data-binary "@${THIS_DIR}/../demos/ClientComplaints/yaml/processor_complaint_insight_generator.yml" | jq .
	 curl -s -X POST ${URL}/v2/catalog/processors -H "authorization: Bearer ${JWT}" -H 'content-type: text/yaml' --data-binary "@${THIS_DIR}/../demos/ClientComplaints/yaml/processor_complaint-classifier.yml" | jq .
	 # curl -s -X POST ${URL}/v2/catalog/processors -H "authorization: Bearer ${JWT}" -H 'content-type: text/yaml' --data-binary "@${THIS_DIR}/../demos/ClientComplaints/yaml/processor_insight_retriever.yml" | jq .	 
	 
agent.createSnapshot:
	curl -s -X POST ${URL}/v2/agents/snapshots -H "authorization: Bearer ${JWT}" -H 'content-type: application/json' -d '{"agentName": "client-complaints-agent", "tags": [ "test" ], "version": 1}' | jq .
	
agent.getTargets:
	curl -s -X GET ${URL}/v2/agents/targets -H "authorization: Bearer ${JWT}" -H 'content-type: application/json' | jq .

agent.createInstance:
	# get snapshot id from the insight_generator_agent createSnapshot command
	# get environment id from getTargets
	curl -s -X POST ${URL}/v2/agents/instances -H "authorization: Bearer ${JWT}" -H 'content-type: application/json' -d '{"snapshotId": "5a1f0e6614b9200001cf9949", "environmentId": "5a1f0a1b7c2e240001142805"}' | jq .

agent.executeService:
	# use the instance id from createInstance target
	curl -s -X POST ${URL}/v2/agents/services/5a1f107e14b9200001cf994a/input -H "authorization: Bearer ${JWT}" -H 'content-type: application/json' -d '{"typeName": "cortex/Text", "body" : {"text" : "This is a test."}}' | jq .

connections.bootstrap:
	curl -s -X GET ${URL}/v2/connections/_/bootstrap -H "authorization: Bearer ${JWT}"| jq .

connections.getTypes:
	curl -s -X GET ${URL}/v2/connections/types -H "authorization: Bearer ${JWT}" | jq .

catalog.bootstrap:
	curl -s -X GET ${URL}/v2/catalog/_/bootstrap -H "authorization: Bearer ${JWT}"| jq .

create.exampleData:
	mongo localhost:27017/example bootstrap_example_data.js
	
cc.bootstrap: tenant.register user.register catalog.bootstrap connections.bootstrap catalog.saveTypes

cc.setup: catalog.saveProcessor catalog.createAgent

taai.execute: agent.createSnapshot agent.getTargets

#test: register catalog.createAgent catalog.saveProcessor agent.createSnapshot connections.bootstrap connections.getTypes catalog.bootstrap datasets.bootstrap
