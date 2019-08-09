# Service Template for Lookup/Reference data services
The idea is to create single service will be used for all the lookups or reference data.
This service will return a behaviour subject for every lookup data model. 
Below code snippets have details of CRUD operations on these lookup data models.
The benefit is all data will be cached in a subject and using the subject same data can be retrieved. This should avoid repetive database calls.

Below code snippets outline the service templates. Replace `GenericModel` to your lookup/reference data model `GenericDataService` to your lookup service.

Create your lookup or reference data model
```typescript
export class GenericModel {
  id:string;
  field:string;
}
```

Import your model
```typescript
import { GenericModel } from './generic.model';
```

Create the service. Use single service for all the lookup/reference data.
```typescript
@Injectable()
export class GenericDataService {
```
  Create a new subject for each reference data or lookup. Add more subjects here for different lookup data or reference data.
  ```typescript
  private genericModelSubject: BehaviorSubject<GenericModel[]>;
  ```  
  
  Add urls and endpoints for each.
  ```typescript
  private genericModelEndpoint: string = API_ENDPOINT_CONFIG.genericModelUrlFromEnvironmentTs + '/<endpoint>';
  ```
  
  Create boolean to check if subject for generic-model is initialized.
  ```typescript
  private genericModelInitialized = false;
  ```
  
  Use ServiceHttpClient as base class - this will do session extension/error handling.
  Use GrowlService to display grown notifications to user.
  ```typescript
  constructor(
    private http: ServiceHttpClient,
    private growlService: GrowlService,
    ) {
    /////Initialize all the subjects
    this.genericModelSubject = (BehaviorSubject<GenericModel[]>)new BehaviorSubject([]);
    
    /////Initialize all the flags
    this.genericModelInitialized = false;
  }
  ```

  Create get method to fetch all the values that return the subject.
  ```typescript
  getAllGenericModel(): BehaviorSubject<GenericModel[]> {
    if(!this.genericModelInitialized) {
        this.http.get(this.genericModelEndpoint).subscribe(
          (data: GenericModel[]) => {
            this.genericModelInitialized = true;
            this.genericModelSubject.next(Object.assign({}, data));
          }, error => {
             this.handleError(error);
          }
        );
    }
    return this.genericModelSubject;
  }
  ```
  
  Search function over lookup data.
  ```typescript
  searchGenericModel(genericModelId: string): GenericModel {
    if(!this.genericModelInitialized) {
        return null;
    }
    const genericModelList: GenericModel[] = this.genericModelSubject.getValue();
    var result = genericModelList.filter(function(item) {
        return item.id === genericModelId;
    })[0];
    return result; 
  }
  ```
  
  Create method to add lookup/reference data. This is optional. This method returns the lookup/reference.
  ```typescript
  postGenericModel(genericModelObject: GenericModel): BehaviorSubject<GenericModel>[] {
    this.http.post(this.genericModelEndpoint, JSON.stringify(genericModelObject)).subscribe(
      (data => {
        const genericModelList: GenericModel[] = this.genericModelSubject.getValue();
        
        ////This can be replaced with Array.concat() if instead of object an object array is passed
        genericModelList.push(genericModelObject);
        this.genericModelSubject.next(Object.assign({}, genericModelList));
        return this.genericModelSubject;
      }, error => {
        this.handleError(error);
      }
    );
  }
  ```
  In above highlighted snippet you can replace `Array.push()` to `Array.concat()` if lookup/reference object to object array. 
  ```typescript
  genericModelList.concat(genericModelObject);
  ```
  
  Update lookup/reference data. Return subject after updating.
  ```typescript
  putGenericModel(genericModelObject: GenericModel): BehaviorSubject<GenericModel[]> {
    this.http.put(this.genericModelEndpoint, JSON.stringify(genericModelObject)).subscribe(
      (data: GenericModel) => {
        const genericModelList: GenericModel[] = this.genericModelSubject.getValue();
        genericModelList.forEach((model, i) => {
          if (model.id === data.id) {
            genericModelList[i] = data;
            this.genericModelSubject.next(Object.assign({}, genericModelList));
            return this.genericModelSubject;
          }
        });

      }, error => {
        this.handleError(error);
      }
    );
  }
  ```

  Delete lookup/reference data, update the subject post that and return subject.
  ```typescript
  deleteGenericModel(genericModelId: string): BehaviorSubject<GenericModel[]> {
    this.http.delete(this.genericModelEndpoint + `/${<genericModelId>}`).subscribe(
      (data: GenericModel) => {
        const genericModelList: GenericModel[] = this.genericModelSubject.getValue();
        genericModelList.forEach((model, i) => {
          if (model.id === data.id) {
            genericModelList.splice(i, 1);
            this.genericModelSubject.next(Object.assign({}, genericModelList));
            return this.genericModelSubject;
          }
        });
      }, error => {
        this.handleError(error);
      }
    );
  }  
  ```  
  Use handleError method to handle the error. Most of service errors are shown to user in Growl notifications.
  ```typescript
  handleError(error: any): void {
    // handle error as per business requirements
    if (error.status === constants.responseStatusCode.notFound) {
    // do something on notFound
        this.growlService.showErrorNotification(error.message, error.title, false);
    }
    else { // else if you want component to handle the error
        Observable.throw(error);
    }
  }
 
 ```   


In your component file add Inject `<generic-service>`.
```typescript
export class ExampleComponent implements OnInit {
  genericModelSubject: BehaviorSubject<GenericModel[]>;
  /////or
  genericModelsList:GenericModel[]
  constructor(
    private formBuilder: FormBuilder,
    private genericServiceObject: GenericDataService) {
  }
``` 
  

   Add ngOnInit method to initialize the lookup data
   ```typescript
   ngOnInit() {
     this.genericModelSubject =  this.genericServiceObject.getAllGenericModel();
     
     /////another way to access the service is
     this.genericDataService.getAllGenericModel().subscribe(
      data => {
        this.genericModelsList = data;
        ////below code shows how single model can be searched
        this.singleGenericModel = this.genericDataService.searchGenericModel(2);
      }
    )
   }
   ```
   
   Below snippet shows create methods to add lookup/reference
   ```typescript
   create() {
    let genericModelObject: GenericModel;
    genericModelObject = {
        id:<value>;
        field:<value>;
    }
    this.genericModelSubject =  this.genericServiceObject.post(genericModelObject);
   }
   ```
   
   Below snippet shows update method to add lookup/reference
   ```typescript
   update() {
    let genericModelObject: GenericModel;
    genericModelObject = {
        field:<updatedValue>;
    }
    this.genericModelSubject =  this.genericServiceObject.put(genericModelObject);
   }
   ```   
  
In your template example-component.html file below line gives an exammple to use subjects.
```
<div *ngFor="let genericModel of genericServiceObject.genericModelSubject | async">
{{ genericModel.id }} {{ genericModel.value }}
</div>
``` 

Or if you define array of lookup/reference models in your component, then you can use below code.
```
<div *ngFor="let genericModel of genericModelsList">
{{ genericModel.id }} {{ genericModel.value }}
</div>   
```
