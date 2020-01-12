# -import  { chain ,  Rule ,  SchematicContext ,  SchematicsException ,  Tree }  из  '@ angular-devkit / schematics' ;
import  { addPackageToPackageJson ,  getPackageVersionFromPackageJson }  из  './package-config' ;
import  { Schema }  из  './schema' ;
import  { scullyVersion ,  scullyComponentVersion }  из  './version-names' ;
импортировать  { NodePackageInstallTask ,  RunSchematicTask }  из  '@ angular-devkit / schematics / tasks' ;
import  { getSourceFile ,  getSrc }  из  '../utils/utils' ;
import  { addImportToModule ,  insertImport }  из  '@ schematics / angular / utility / ast-utils' ;
импортировать  { InsertChange }  из  '@ schematics / angular / utility / change' ;
импортировать * как  ts  из  '@ schematics / angular / third_party / github.com / Microsoft / TypeScript / lib / typescript' ;

экспорт по  умолчанию  ( параметры : схема ) : правило  =>  {
  обратная  цепь ( [
    addDependencies ( параметры ) ,
    importHttpClientModule ( параметры ) ,
    addHttpClientModule ( параметры ) ,
    addPolyfill ( параметры ) ,
    injectIdleService ( параметры ) ,
    runScullySchematic ( параметры ) ,
  ] ) ;
} ;

const  addDependencies  =  ( параметры : схема )  =>  ( дерево : дерево ,  контекст : SchematicContext )  =>  {
  addPackageToPackageJson ( tree ,  '@ scullyio / scully' ,  ` $ { scullyVersion } ` ) ;
  const  ngCoreVersionTag  =  getPackageVersionFromPackageJson ( tree ,  '@ angular / core' ) ;
  if  ( + ngCoreVersionTag . search ( / ( \ ^ 8 | ~ 8 ) / g )  ===  0 )  {
    консоль . log ( 'Установить ng-lib для Angular v8' ) ;
    addPackageToPackageJson ( tree ,  '@ scullyio / ng-lib-8' ,  ` $ { scullyComponentVersion } ` ) ;
  }  еще  {
    консоль . log ( 'Установить ng-lib для Angular v9' ) ;
    addPackageToPackageJson ( tree ,  '@ scullyio / ng-lib' ,  ` $ { scullyComponentVersion } ` ) ;
  }
  контекст . регистратор . info ( '✅️ Добавленная зависимость' ) ;
} ;

const  importHttpClientModule  =  ( параметры : схема )  =>  ( дерево : дерево ,  контекст : SchematicContext )  =>  {
  попытаться  {
    const  mainFilePath  =  `./ $ { getSrc ( tree ) } / app / app.module.ts` ;
    const  рекордер  =  дерево . beginUpdate ( mainFilePath ) ;

    const  mainFileSource  =  getSourceFile ( tree ,  mainFilePath ) ;
    const  importChange  =  insertImport (
      mainFileSource ,
      mainFilePath ,
      'HttpClientModule' ,
      @ Угловой / общий / HTTP '
    )  как  InsertChange ;
    if  ( importChange . toAdd )  {
      записывающее устройство . insertLeft ( importChange . pos ,  importChange . toAdd ) ;
    }
    дерево . commitUpdate ( устройство записи ) ;
    возвратное  дерево ;
  }  catch  ( e )  {
    консоль . log ( 'ошибка при импорте httpclient' ,  e ) ;
  }
} ;

const  addHttpClientModule  =  ( параметры : схема )  =>  ( дерево : дерево ,  контекст : SchematicContext )  =>  {
  const  mainFilePath  =  `./ $ { getSrc ( tree ) } / app / app.module.ts` ;
  константный  текст  =  дерево . читать ( mainFilePath ) ;
  if  ( text  === null )  {
    выбросить  новое  SchematicsException ( `File $ { mainFilePath } не существует` ) ;
  }
  const  sourceText  =  текст . toString ( ) ;
  постоянный  источник  =  ц . createSourceFile ( mainFilePath ,  sourceText ,  ts . ScriptTarget . Latest ,  true ) ;
  const  changes  =  addImportToModule ( source ,  mainFilePath ,  'HttpClientModule' ,  '@ angular / common / http' ) ;
  const  рекордер  =  дерево . beginUpdate ( mainFilePath ) ;
  для  ( сопзЬ  изменения  от  изменений )  {
    if  ( измените  instanceof  InsertChange )  {
      записывающее устройство . insertLeft ( изменить . pos ,  изменить . toAdd ) ;
    }
  }
  дерево . commitUpdate ( устройство записи ) ;
  возвратное  дерево ;
} ;

const  addPolyfill  =  ( параметры : схема )  =>  ( дерево : дерево ,  контекст : SchematicContext )  =>  {
  пусть  polyfills  =  дерево . читать ( ` $ { getSrc ( tree ) } / polyfills.ts` ) . toString ( ) ;
  if  ( polyfills . includes ( 'SCULLY IMPORTS' ) )  {
    контекст . регистратор . info ( '⚠️ Пропуск polyfills.ts' ) ;
  }  еще  {
    polyfills  =
      полифилы  +
      `\ П / ********************************************** ************************************************** ***
* СКАЛЛИ ИМПОРТ
* /
// tslint: disable-next-line: align
импорт 'zone.js / dist / task-tracking'; ` ;
    дерево . перезаписать ( ` $ { getSrc ( tree ) } / polyfills.ts` ,  polyfills ) ;
  }
} ;

const  injectIdleService  =  ( параметры : схема )  =>  ( дерево : дерево ,  контекст : SchematicContext )  =>  {
  попытаться  {
    const  appComponentPath  =  ` $ { getSrc ( tree ) } / app / app.component.ts` ;
    const  appComponent  =  дерево . читать ( appComponentPath ) . toString ( ) ;
    if  ( appComponent . includes ( 'IdleMonitorService' ) )  {
      контекст . регистратор . info ( `⚠️️ Пропуск $ { appComponentPath } ` ) ;
    }  еще  {
      const  idleImport  =  `import {IdleMonitorService} из '@ scullyio / ng-lib';` ;
      // добавлять
      const  idImport  =  ` $ { idleImport } \ n $ { appComponent } ` ;
      const  idle  =  'private idle: IdleMonitorService' ;
      let  output  =  '' ;
      // проверяем, существует ли
      if  ( idImport . search ( / constructor / )  ===  - 1 )  {
        // добавляем, если не существует конструктор
        const  add  =  `\ n конструктор ( $ { idle } ) {} \ n` ;
        константное  положение  =
          idImport . search ( / export class AppComponent {/ g )  +  'экспортный класс AppComponent {' . длина ;
        выход  =  [ idImport . срез ( 0 ,  позиция ) ,  добавить ,  idImport . срез ( позиция ) ] . присоединиться ( '' ) ;
      }  еще  {
        const  coma  =  haveMoreInjects ( idImport ) ;
        const  add  =  ` $ { idle } $ { coma } ` ;
        if  ( idImport . search ( / constructor \ (/ )  ===  - 1 )  {
          Const  положение  =  idImport . поиск ( / конструктор \ (/ g )  +  'конструктор (' . длина ;
          выход  =  [ idImport . срез ( 0 ,  позиция ) ,  добавить ,  idImport . срез ( позиция ) ] . присоединиться ( '' ) ;
        }  еще  {
          Const  положение  =  idImport . поиск ( / конструктор \ (/ g )  +  'конструктор (' . длина ;
          выход  =  [ idImport . срез ( 0 ,  позиция ) ,  добавить ,  idImport . срез ( позиция ) ] . присоединиться ( '' ) ;
        }
      }
      дерево . перезаписать ( appComponentPath ,  выход ) ;
    }

    function  haveMoreInjects ( fullComponent : string )  {
      const  match  =  '(([^ ()] * (private | public) [^ ()] *))' ;
      if  ( fullComponent . search ( match ) ! == - 1 )  {
        вернуть  ',' ;
      }
      возврат  '' ;
    }
  }  catch  ( e )  {
    консоль . журнал ( «ошибка в режиме ожидания» ) ;
  }
} ;

const  runScullySchematic  =  ( параметры : схема )  =>  ( дерево : дерево ,  контекст : SchematicContext )  =>  {
  const  nextRules : Rule [ ]  =  [ ] ;
  if  ( options . blog  ===  true )  {
    // @ ts-ignore
    nextRules . push ( context . addTask ( new  RunSchematicTask ( «блог» ,  опции ) ,  [ ] ) ) ;
  }
  // tslint: disable-next-line: no-shadowed-variable
  nextRules . push ( ( дерево : дерево ,  контекст : SchematicContext )  =>  {
    const  installTaskId  =  context . addTask ( новый  NodePackageInstallTask ( ) ) ;
    контекст . addTask ( новый  RunSchematicTask ( «запустить» ,  опции ) ,  [ installTaskId ] ) ;
  } ) ;

  возвратная  цепочка ( nextRules ) ;
} ;
