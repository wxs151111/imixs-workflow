#The WorkflowService Interface
The WorkflowService is the Java EE Implementation if the   {{{http://www.imixs.org/api/workflowmanager.html}WorkflowManager}} which can be used  in any Java enterprise application. The component allows you to process, update and find workItems. 

Before a workitem can be processed by the WorkflowService a workflow model  need to be available on the workflow server. To create a model and upload it to the workflow server
  the {{{http://www.imixs.org/modeler/}Imixs-Workflow Modeler}}  can be used. The following example shows how a workitem can be processed using the WorkflowService.  A workitem must provide at least the properties "$ProcessID" and "$ActivityID" to indicate which  workflow activity should be processed by the workflowManager. So the $ProcessID and the $ActivityID must point to an existing model entry provided by the workflow model. 

	  @EJB
	  org.imixs.workflow.jee.ejb.WorkflowService workflowService;
	  //...
	  // create an empty workitem
	  ItemCollection workitem=new ItemCollection();
	  workitem.replaceItemValue("type", "workitem");
	  workitem.replaceItemValue("name", "Anna");
	  workitem.replaceItemValue("txtTitel", "My first workflow example");
			
	  // set workflow status based on a supported model
	  workitem.replaceItemValue("$processID", 10);
	  workitem.replaceItemValue("$ActivityID", 10);
	  // process the worktiem
	  workitem=workflowService.processWorkItem(workitem);

The workitem is now controlled by the workflow Manager. So depending on the Workflow Model definition there a different ways to access a worktiem processed by the WorkflowService. To get the current list of all workitems created by the current user, you can call the  method() getWorkListByCreator. 
  
	  @EJB
	  org.imixs.workflow.jee.ejb.WorkflowService workflowService;
	  //...
	  Collection<ItemCollection> statuslist=workflowService.getWorkListByCreator(null,0,-1);
	  //...

  
To call the list of all workitems created by a specific user, you need to provide the username/userid.
  
    Collection<ItemCollection> statuslist=workflowService.getWorkListByCreator('Manfred',0,-1);
  
You can also use a paging mechanism to browse through long result sets. The following example
shows how to get 5 workitems starting at the tenth record
  
    Collection<ItemCollection> statuslist=workflowService.getWorkListByCreator(null,10,5);

##Worklist methods
The following list of getWorkList methods shows how workitems can be accessed by different categories. The workflowService returns only workitems in a worklist if the user has read access to the workitems. If a workitem is not access able for the user this workitem will be hidden from the list.  The result sets can be ordered by modified or creation date. 

###getWorkList
Returns a collection of workitems for the current user. A Workitem belongs to a user or role if the  user has at least read write access to this workitem. 

    Collection<ItemCollection> list=workflowService.getWorkList();
    //...

You can specify the type, the start position, the count and sort order of workitems returned 
by this method. The type of a workitem is defined by the workitem proprety 'type' which can be set before a workitem is processed.

	  String type="workitem";
	  Collection<ItemCollection> list=workflowService.getWorkList(0,-1,
	     type,WorkflowService.SORT_ORDER_CREATED_DESC);
	  //...

###getWorkListByAuthor

Returns a collection of workitems belonging to a specified user. This filter can be set to 
 a username or a user role defined by the application. A Workitem belgons to a user or role if the  user has write access to this workitem. So the method returns workitems which can be 
 processed by the user.  

	  String type="workitem";
	  String user="Manfred"
	  Collection<ItemCollection> list=workflowService.getWorkList(user,0,-1,
	     type,WorkflowService.SORT_ORDER_CREATED_DESC);
	  //...


###getWorkListByGroup
Returns a collection of workitems belonging to a specified workflow group.  The workflow group is defined by the workflow model and includes all process entities defined by 
 a business process 

	  String type="workitem";
	  String group="Ticketservice";
	  Collection<ItemCollection> list=workflowService.getWorkListByGroup(group,0,-1,
	     type,WorkflowService.SORT_ORDER_CREATED_DESC);
	  //...


###getWorkListByProcessID

Returns a collection of workitems belonging to a specified $processID defined by the workflow model.

	  String type="workitem";
	  Collection<ItemCollection> list=workflowService.getWorkListByProcessID(2100,0,-1,
	     type,WorkflowService.SORT_ORDER_CREATED_DESC);
	  //...


###getWorkListByOwner
Returns a collection of workitems containing a namOwner property belonging to a specified username.  The namOwner property is typical controlled by the OwnerPlugin using the Imixs Workflow Modeler

	  String type="workitem";
	  String user="Manfred"
	  Collection<ItemCollection> list=workflowService.getWorkListByOwner(user,0,-1,
	     type,WorkflowService.SORT_ORDER_CREATED_DESC);
	  //...
  
###getWorkListByWriteAccess
Returns a collection of workitems where the current user has at least writeAccess. This means the either the  username or one of the user roles is contained in the $writeaccess property of each workitem returned by the method.
 
	  String type="workitem";
	  Collection<ItemCollection> list=workflowService.getWorkListByWriteAccess(0,-1,
	     type,WorkflowService.SORT_ORDER_CREATED_DESC);
	  //...
  
###getWorkListByRef
 Returns a collection of workitems belonging to a specified workitem identified by the attribute $UniqueIDRef. 

	  String type="workitem";
	  Collection<ItemCollection> list=workflowService.getWorkListByRef(refID,0,-1,
	     type,WorkflowService.SORT_ORDER_CREATED_DESC);
	  //...
  
##Model Version Management 
Each time a running process instance is updated, the WorkflowService compares  the internal model version with the model versions provided by the model repository.  In case the current model version is no longer available the WorkflowService  automatically upgrades an active process instance to the latest version in the   repository. Therefore the engine verifies the task ID (numprocessid) and the process name   (txtworkflowgroup) with the corresponding models. This mechanism allows to upgrade  process instances at run time to a newer version. 
  
It is also possible to handle different versions of a model at the same time.   In this case each process instance is processed by the model version from which  it was started.
  
 