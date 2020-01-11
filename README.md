[Markdown:Syntax](https://daringfireball.net/projects/markdown/syntax)
[curso](https://github.com/kaikcreator/angular-school-c5.git)


## Formularios basados en Template


### 7.4 Control de estado  y feedhback visual
template reference variable (#)  #nameControl, hace referencia a un objeto.

> Todo elemento HTML tiene una propiedad ClassName que devuelve un texto con las clases css que se estan aplicando sobre el mismo.

* ng-untouched  : Todavia no se ha visitado el elemento.
* ng-pristine   : Indica que nunca e cambiado el valor del campo.
* ng-invalid    : Indica que es invalido.

Lo contrario:

* ng-touched : Indica que e visitado el elemento.
* ng-dirty   : Indica que e cambiado el valor.
* ng-valid   : Indica que es valido.


### 8.5 Mensaje de Error

#nameControl="ngModel", hace referencia a un objeto del tipo ngModel y tiene sus propiedades.

```html
    <div class="form-item">
        <label for="name">Name:</label>
        <input type="text" id="name" [(ngModel)]="model.name" name="name" required minlength="2" #nameControl="ngModel">
        <ng-container *ngIf="!nameControl.valid && nameControl.touched">
            <p *ngIf="nameControl.errors.required" class="error-message">This is field is required</p>
            <p *ngIf="nameControl.errors.minlength" class="error-message">A name needs at least 2 characters!!</p>
        </ng-container>
    </div>
```

```css
.ng-invalid:not(form).ng-touched{
    border-color: red;
}

.ng-valid:not(form)[required]{
    background: url("assets/check.png");
    background-position: right;
    background-repeat: no-repeat;
}
```

### 9.6 ngForm, Dehabilitar el boton de submit y reset

Podemos usar ngForm para volver a su estado inicial el formulario de 2 maneras:
#### 1. Html
```html
<form (ngSubmit)="addContact(); contactForm.reset()">
 
</form>
```
#### 2. Ts
```ts
import { NgForm } from '@angular/forms';
...
@ViewChild('contactForm', {static:true}) contactForm:NgForm;
...
this.contactForm.reset();
```

#### Deshabilita el submit si el formulario no es valido
```html
<form (ngSubmit)="addContact();" #contactForm="ngForm">
    <button class="form-button" type="submit" [disabled]="!contactForm.valid">Add contact</button>
</form>
```

### Validadores

Angular proporciona directivas de validacion como: required, minlength, pattern... etc.

> [Directivas de validacion de angular](https://angular.io/api/forms/Validators)

> [Sobre expresiones regulares](https://regexr.com/)

### Validadores personalizados

La directiva es como un componente, pero sin template, es un conjunto de logica que añades a otros elementos y al igual que los componentes las directivas tienen un selector que es el nombre con el que se la llama desde un elemento.

Como los componentes las directivas aceptan providers para la inyeccion de dependencias.

```ts
@Directive({
 selector: '[startsWithCapital]',
 providers: [{  provide: NG_VALIDATORS,
                useExisting: StartsWithCapitalValidatorDirective,
                multi: true
            }]
})
```
> provide:
> useExisting : Indica que se tiene que usar la instancia concreta del validador.
> multi : para añadir este validador al resto de los NG_VALIDATORS

#### Paso 1: Crear la directive "startsWithCapital"
* Se crea la directiva que:
exporta una factoria que devuelve la funcion de validacion
exporta una directiva que utiliza esa funcion de validacion.

> Esto para poder aprovechar los formularios reactivos.

#### Paso 2: Agregar al module del componente.
```ts
    declarations:[
        ContactDetailComponent,
        ContactDetailShellComponent,
        ContactFormComponent,
        StartsWithCapitalValidatorDirective
    ]
```
#### Paso 3: Utilizar 
```ts
        <input type="text" id="name" [(ngModel)]="model.name" name="name" 
            required minlength="2" [startsWithCapital]="true" #nameControl="ngModel">
        <ng-container *ngIf="!nameControl.valid && nameControl.touched">
            <p *ngIf="nameControl.errors.startsWithCapital" class="error-message">
                A name should start with capital letter!!
            </p>               
        </ng-container>
```



## FOMULARIOS REACTIVOS

>[Documentacion angular FormControl](https://angular.io/api/forms/FormControl)

### FormControl
Te permite controlar un campo de entrada.

#### Paso 1 Agregar al .module de la app.

```ts
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
    imports: [
        ReactiveFormsModule

    ],
    declarations:[
    ]
})
```
#### Paso 2 Usar en el formulario de la clase componente
```ts
export class ContactFormComponent implements OnInit {
  //   propiedad
  public name:FormControl = new FormControl('');

  ngOnInit() {
    this.name.setValue('Algo'); // valor por defecto
  }

}
```
Desde el template se hace el binding haciendo referencia a la propiedad del componente.
```html  
<input type="text" id="name" [formControl]="name">
<p>Name is: {{name.value}}</p>
```
> Solo se puede utilizar en un template plano, sin asociar una etiqueta <form></form> para ello es necesario usar un FormGroup para poder usar los formularios reactivos.


### FormGroup
Te permite controlar un conjunto de campos de entrada de un <form></form>

#### Paso 1 Agregar la directiva en el componente.
```ts
import { FormControl, FormGroup } from '@angular/forms';

export class ContactFormComponent implements OnInit {

  // propied
  public contactForm:FormGroup = new FormGroup({
    name: new FormControl(''),
    email: new FormControl(''),
    address: new FormControl('')
  });
}
```
#### Paso 2 Hacer el binding del template
```html
<form [formGroup]="contactForm">   
        <!-- name field -->
    <div class="form-item">
        <label for="name">Name:</label>
        <input type="text" id="name" formControlName="name">
    </div>
</form>
```


### Reactivo grupos anidados
Resumen: Utilizar la clase FormGroup y definir en su interior los elementos con su FormControl
en el template es indicarle al formulario que elemento es el que contiene el elemento anidado y 
luego hacer el binding de sus elementos.

Los formularios reactivos es que el estado es inmutable, y se pueden aceder a sus elementos de forma asincrona.


#### Paso 1 Agregar la directiva en el componente.
```ts
import { FormControl, FormGroup } from '@angular/forms';

export class ContactFormComponent implements OnInit {

  // propied
  public contactForm:FormGroup = new FormGroup({
    name: new FormControl(''),
    phone: new FormGroup({
      type: new FormControl(''),
      number: new FormControl('')
    }),
    email: new FormControl(''),
    address: new FormControl('')
  });
}
```
#### Paso 2 Hacer el binding del template con elementos anidaddos
```html
<form [formGroup]="contactForm">   
        <!-- name field -->
    <div class="form-item">
        <label for="name">Name:</label>
        <input type="text" id="name" formControlName="name">
    </div>

    <!-- phones field -->
    <div class="form-item" formGroupName="phone">
        <label for="phone-type">Phone</label>
        <select id="phone-type" formControlName="type">
            <option [value]="type" *ngFor="let type of phoneTypes">{{type}}</option>
        </select>
        <input type="tel" id="phone-number" formControlName="number">
        <!-- <p class="form-action" (click)="addNewPhoneToModel()">Add phone +</p> -->
    </div>
</form>
```


### INMUTABILIDAD Y FILE INPUT
PatchValue: realiza una actualizacion parcial de los elementos del formulario.
esto para no volver a construir el objeto pasando cada uno de los datos del formulario.

```html
<form [formGroup]="contactForm">    
    <div class="form-item-image">
        <img [src]="contactForm.value.picture">
        <input type="file" accept=".png,.jpg,.jpeg" (change)=addImage($event)>
    </div>
</form>
```

```ts
  public contactForm:FormGroup = new FormGroup({
    name: new FormControl(''),
    picture: new FormControl('assets/default-user.png'),
    phone: new FormGroup({
      type: new FormControl(null),
      number: new FormControl('')
    }),
    email: new FormControl(''),
    address: new FormControl('')
  });

addImage(event){
    const file = event.target.files[0];
    var reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = (evt) => {
      this.contactForm.patchValue({
        picture: reader.result
      });
    }
  }
```

### REACTIVOS VALIDACION
RESUMEN: simplemente a;adir como un elemento mas del FormControl

```ts
export class ContactFormComponent implements OnInit {

  public contactForm:FormGroup = new FormGroup({
    name: new FormControl('', [Validators.required, Validators.minLength(2), startsWithCapitalValidator()]),
    picture: new FormControl('assets/default-user.png'),
    phone: new FormGroup({
      type: new FormControl(null),
      number: new FormControl('')
    }),
    email: new FormControl(''),
    address: new FormControl('')
  });

  get name(){
    return this.contactForm.get('name');
  }
}
```

```html
<form [formGroup]="contactForm">    
<div class="form-item">
        <label for="name">Name:</label>
        <input type="text" id="name" formControlName="name">
        <ng-container *ngIf="!contactForm.get('name').valid && contactForm.get('name').touched">
            <p *ngIf="contactForm.get('name').errors.required" class="error-message"> This filed is required!!!</p>
            <p *ngIf="contactForm.get('name').errors.minlength" class="error-message">A name needs at least 2 characters!!!</p>
            <p *ngIf="contactForm.get('name').errors.startsWithCapital && !name.errors.minlength" class="error-message"> The name should start with capital letter!!!</p>
        </ng-container>
    </div>
<form [formGroup]="contactForm">    
```
or

```html
<form [formGroup]="contactForm">  
    <!-- name field -->
    <div class="form-item">
        <label for="name">Name:</label>
        <input type="text" id="name" formControlName="name">
        <ng-container *ngIf="!name.valid && name.touched">
            <p *ngIf="name.errors.required" class="error-message"> This filed is required!!!</p>
            <p *ngIf="name.errors.minlength" class="error-message">A name needs at least 2 characters!!!</p>
            <p *ngIf="name.errors.startsWithCapital && !name.errors.minlength" class="error-message"> The name should start with capital letter!!!</p>
        </ng-container>
    </div>
<form [formGroup]="contactForm">  
```

### PROGRAMACION REACTIVA DE FORMULARIOS
ZIP: combina de fomar ordenada los eventos de varios observables.
traera para nuestro en la zip([0] estado, [1]valor)

```ts
  ngOnInit() {
    const contact = localStorage.getItem('contact');
    if(contact){
      this.contactForm.setValue(JSON.parse(contact));
    }

    zip(this.contactForm.statusChanges, this.contactForm.valueChanges).pipe(
      filter(([state, value]) => state == 'VALID'),
      map( ([state, value]) => value),
      tap(data => console.log(data))
    ).subscribe( formValue => {
      localStorage.setItem('contact', JSON.stringify(formValue))
    });
  }
```

### FORMULARIOS REACTIVOS DINAMICOS
1.- FormArray: Para poder tener varios cotroles agrupados.
2.- Iterando los objetos del FormArray para que siempre mantenga su estructura.
3.-

```ts
import { FormControl, FormGroup, Validators, FormArray } from '@angular/forms';
import { startsWithCapitalValidator } from 'src/app/directives/startsWithCapital.directive';
import { zip } from 'rxjs';
import { tap, filter, map } from 'rxjs/operators';

export class ContactFormComponent implements OnInit {

public contactForm:FormGroup = new FormGroup({
    name: new FormControl('', [ Validators.required, Validators.minLength(2), startsWithCapitalValidator() ]),
    picture: new FormControl('assets/default-user.png'),
    phones: new FormArray([
      new FormGroup({
      type: new FormControl(null),
      number: new FormControl('')
    })
    ]),
    email: new FormControl(''),
    address: new FormControl('')
  });

    ngOnInit() {
    const contact = localStorage.getItem('contact');
    if(contact){
      const contactJSON = JSON.parse(contact);
      this.phones.clear();
      for(let i=0; i < contactJSON.phones.length; i++){
        this.addNewPhoneToModel();
      }
      this.contactForm.setValue(contactJSON);
    }

    zip(this.contactForm.statusChanges, this.contactForm.valueChanges).pipe(
      filter(([state, value]) => state == 'VALID'),
      map( ([state, value]) => value),
      tap(data => console.log(data))
    ).subscribe( formValue => {
      localStorage.setItem('contact', JSON.stringify(formValue))
    });
  }

    addNewPhoneToModel(){
    this.phones.push(
      new FormGroup({
        type: new FormControl(null),
        number: new FormControl('')
      })
    );
  }

    get phones(){
    return this.contactForm.get('phones') as FormArray;
  }
}
```

```html
<div class="form-group" formArrayName="phones">
    <p>Phones:</p>
     <!-- phones field -->
    <div class="form-item" *ngFor="let phone of phones.controls; let i=index"  [formGroupName]="i">
        <label for="phone-type">Phone</label>
        <select id="phone-type" formControlName="type">
            <option [value]="type" *ngFor="let type of phoneTypes">{{type}}</option>
        </select>
        <input type="tel" id="phone-number" formControlName="number">
    </div>
        <p class="form-action" (click)="addNewPhoneToModel()">Add phone +</p>
</div>
```

### FORMULARIOS REACTIVOS SUBMIT RESET
1.- Se agregado un event Binding (ngSubmit)=""
2.- Se agredo el contacto del formulario, y se limpio los elementos del FormArray ademas de 
eliminar el objeto del localstorage.


```ts
  addContact(){
    this.contactsService.addContact(this.contactForm.value);
    this.phones.clear();
    this.addNewPhoneToModel();
    this.contactForm.reset({
      picture: 'assets/default-user.png'
    });
    localStorage.removeItem('contact');
  }
```

```html
<form [formGroup]="contactForm" (ngSubmit)="addContact()">    

</form>
```

### FORMULARIO CON FORMBUILDER
Util para evitar escribir de manera verbosa.

* ANTES
```ts
  public contactFormOld:FormGroup = new FormGroup({
    name: new FormControl('', [ Validators.required, Validators.minLength(2), startsWithCapitalValidator() ]),
    picture: new FormControl('assets/default-user.png'),
    phones: new FormArray([ 
      new FormGroup({
        type: new FormControl(null),
        number: new FormControl('')
      })
    ]),
    email: new FormControl(''),
    direction: new FormControl('')
  });
```

* DESPUES CON FORMBUILDER
```ts
import { FormControl, FormGroup, FormArray, Validators, FormBuilder } from '@angular/forms';

  public contactForm = this.fb.group({
    name:['', [Validators.required, Validators.minLength(2), startsWithCapitalValidator()]],
    picture: ['assets/default-user.png'],
    phones: this.fb.array([
      this.fb.group({
        type: [null],
        number: ['']
      })
    ]),
    email: ['', Validators.required],
    direction: ['']
  })
```


