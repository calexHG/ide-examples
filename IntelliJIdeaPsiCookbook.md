

# IntelliJ IDEA PSI Cookbook #

&lt;wiki:gadget url="https://wiki-gadgets.googlecode.com/git/gadgets/ads/banner2.xml" width="740" height="100" border="0" /&gt;



## Reference ##
  * [PSI Helpers and Utilities](http://confluence.jetbrains.com/display/IntelliJIDEA/PSI+Helpers+and+Utilities)
  * [PSI Cookbook](http://confluence.jetbrains.com/display/IDEADEV/PSI+Cookbook)
  * [Project Structure](http://confluence.jetbrains.com/display/IDEADEV/Structure+of+IntelliJ+IDEA+Project)

## General Psi Methods ##

### Query a Class ###
Query a PsiClass using JavaPsiFacade. In this example I'm searching for annotation.

```
// Annotation search. 
private void findSlots() {
    GlobalSearchScope scope = GlobalSearchScope.allScope(project);
    PsiClass psiClass = JavaPsiFacade.getInstance(project).findClass("com.gwtplatform.mvp.client.annotations.ContentSlot", scope);

    slots = new ArrayList<String>();
    Query<PsiMember> query = AnnotatedMembersSearch.search(psiClass, GlobalSearchScope.allScope(project));
    query.forEach(new Processor<PsiMember>() {
        public boolean process(PsiMember psiMember) {
            PsiClass container = psiMember.getContainingClass();
            String classname = container.getName();
            slots.add(classname + "." + psiMember.getName());
            return true;
        }
    });

    String[] listData = new String[slots.size()];
    slots.toArray(listData);
    contentSlotsList.setListData(listData);
}
```

### Find a Class ###
Find a class in the project with a name.
```
GlobalSearchScope scope = GlobalSearchScope.allScope(project);
PsiClass[] classes = PsiShortNamesCache.getInstance(project).getClassesByName("EastModule", scope);
```

### Find All Classes ###
Find all the classes in the project including libraries classes.
```
String[] classes = PsiShortNamesCache.getInstance(project).getAllClassNames();
```


&lt;wiki:gadget url="https://wiki-gadgets.googlecode.com/git/gadgets/ads/banner2.xml" width="740" height="100" border="0" /&gt;


## Java ##

### Finding a Java Class ###
Finding a java PsiClass with GlobalSearchScope.
```
GlobalSearchScope scope = GlobalSearchScope.allScope(project);
PsiClass psiClass = JavaPsiFacade.getInstance(project).findClass("com.gwtplatform.mvp.client.annotations.ContentSlot", scope);
```


## Directories ##

### Roots ###
```
PsiManager psiManager = PsiManager.getInstance(project);
for (VirtualFile root : ProjectRootManager.getInstance(psiManager.getProject()).getContentSourceRoots()) {
	PsiDirectory directory = psiManager.findDirectory(root);
}
```

### Directory to Package ###
```
PsiPackage selectedPackage = JavaDirectoryService.getInstance().getPackage((PsiDirectory) d);
```

## Packages ##

### Find Package ###
```
PsiPackage p = JavaPsiFacade.getInstance(project).findPackage(qualifiedName);
```

### Find a Classes Package ###
```
PsiJavaFile javaFile = (PsiJavaFile) psiClass.getContaningFile();
PsiPackage pkg = JavaPsiFacade.getInstance(project).findPackage(javaFile.getPackageName());
```

### Top Level Packages / Roots ###
Get the top level packages.
```
import com.intellij.ide.projectView.ViewSettings;

public class ProjectViewSettings implements ViewSettings {
    @Override
    public boolean isShowMembers() {
        return false;
    }

    @Override
    public boolean isStructureView() {
        return false;
    }

    @Override
    public boolean isShowModules() {
        return false;
    }

    @Override
    public boolean isFlattenPackages() {
        return false;
    }

    @Override
    public boolean isAbbreviatePackageNames() {
        return false;
    }

    @Override
    public boolean isHideEmptyMiddlePackages() {
        return false;
    }

    @Override
    public boolean isShowLibraryContents() {
        return false;
    }
}
```

```
private List<PsiPackage> getPackages() {
    Project myProject = presenterConfigModel.getProject();
    ProjectViewSettings viewSettings = new ProjectViewSettings();

    final List<VirtualFile> sourceRoots = new ArrayList<VirtualFile>();
    final ProjectRootManager projectRootManager = ProjectRootManager.getInstance(myProject);
    ContainerUtil.addAll(sourceRoots, projectRootManager.getContentSourceRoots());

    final PsiManager psiManager = PsiManager.getInstance(myProject);
    final List<AbstractTreeNode> children = new ArrayList<AbstractTreeNode>();
    final Set<PsiPackage> topLevelPackages = new HashSet<PsiPackage>();

    for (final VirtualFile root : sourceRoots) {
        final PsiDirectory directory = psiManager.findDirectory(root);
        if (directory == null) {
            continue;
        }
        final PsiPackage directoryPackage = JavaDirectoryService.getInstance().getPackage(directory);
        if (directoryPackage == null || PackageUtil.isPackageDefault(directoryPackage)) {
            // add subpackages
            final PsiDirectory[] subdirectories = directory.getSubdirectories();
            for (PsiDirectory subdirectory : subdirectories) {
                final PsiPackage aPackage = JavaDirectoryService.getInstance().getPackage(subdirectory);
                if (aPackage != null && !PackageUtil.isPackageDefault(aPackage)) {
                    topLevelPackages.add(aPackage);
                }
            }
            // add non-dir items
            children.addAll(ProjectViewDirectoryHelper.getInstance(myProject).getDirectoryChildren(directory, viewSettings, false));
        } else {
            // this is the case when a source root has package prefix assigned
            topLevelPackages.add(directoryPackage);
        }
    }

    return new ArrayList<PsiPackage>(topLevelPackages);
}
```

### AnAction Package ###
```
public void actionPerformed(AnActionEvent e) {
        Project project = e.getProject();

        PsiElement element = e.getData(LangDataKeys.PSI_ELEMENT);

        PsiPackage selectedPackage = null;
        if (element instanceof PsiClass) {
            PsiClass clazz = (PsiClass) element;
            PsiJavaFile javaFile = (PsiJavaFile) clazz.getContainingFile();
            selectedPackage = JavaPsiFacade.getInstance(project).findPackage(javaFile.getPackageName());
        } else if (element instanceof PsiDirectory) {
            selectedPackage = JavaDirectoryService.getInstance().getPackage((PsiDirectory) e);
        }
    //...
}
```

&lt;wiki:gadget url="https://wiki-gadgets.googlecode.com/git/gadgets/ads/banner2.xml" width="740" height="100" border="0" /&gt;


## Creation ##

### Java Class ###
```
PsiDirectory dir = null;
String name = "";
PsiClass psiClass = JavaDirectoryService.getInstance().createClass(dir, name);
```

### Add Class to Directory ###
```
final PsiDirectory[] createdNameTokensPackageDirectories = createdNameTokensPackage.getDirectories();
final PsiFile element = PsiFileFactory.getInstance(project).createFileFromText(nameFile, JavaFileType.INSTANCE, contents);
ApplicationManager.getApplication().runWriteAction(new Runnable() {
    public void run() {
        PsiElement createdNameTokensClass = createdNameTokensPackageDirectories[0].add(element);
        PsiJavaFile javaFile = (PsiJavaFile) createdNameTokensClass;
        PsiClass[] clazzes = javaFile.getClasses();
    }
});
```

### Java Class and File ###
```
PsiDirectory[] createdNameTokensPackageDirectories = createdNameTokensPackage.getDirectories();
PsiElementFactory factory = JavaPsiFacade.getInstance(project).getElementFactory();
PsiClass nameTokensClass = factory.createClassFromText(contents, null).getInnerClasses()[0];
PsiFile nameTokensFile = createdNameTokensPackageDirectories[0].createFile(nameFile);
nameTokensFile.add(nameTokensClass);
```

### Java Package ###
PackageUtil has a chooser that pops up, so what I do is copy the class, comment that out and pass along the baseDir to the directory. My class looks liek PackageUtilExt.find... Feature request!!!

```
PsiDirectory psiDir = PackageUtil.findOrCreateDirectoryForPackage(module, packageName, baseDir, false, false);
PsiPackage psiPackage = JavaDirectoryService.getInstance().getPackage(psiDir);
```

### Java Field ###
```
PsiElementFactory elementFactory = PsiElementFactory.SERVICE.getInstance(project);
PsiType psiTypeString = elementFactory.createTypeFromText(CommonClassNames.JAVA_LANG_STRING, null);
elementFactory.createField("myField", psiTypeString);
```

### Write Field and Method to Class ###
```
String fieldSource = "public static final String abc = \"abc\";";
String methodSource = "public static String getAbc() {return abc;}";

// add contents to class
PsiElementFactory elementFactory = PsiElementFactory.SERVICE.getInstance(project);
final PsiField newField = elementFactory.createFieldFromText(fieldSource, null);
final PsiMethod newMethod = elementFactory.createMethodFromText(methodSource, null);

ApplicationManager.getApplication().runWriteAction(new Runnable() {
    @Override
    public void run() {
        nameTokensPsiClass.add(newField);
        nameTokensPsiClass.add(newMethod);
    }
});
```

## Utility ##

### Get the Project ###
Getting the project from an action. This handler gives you the AnActionEvent and the project can be retreived from that. This is useful for the other PSI methods.

```
public void actionPerformed(AnActionEvent e) {
    Project project = e.getProject();
}
```

### Get the PsiClass or PsiDirectory ###
Get the psi class or directory.
```
 public void actionPerformed(AnActionEvent e) {
        PsiElement element = e.getData(LangDataKeys.PSI_ELEMENT);
        if (element instanceof PsiClass) {
            System.out.println("isClass");
        } else if (element instanceof PsiDirectory) {
            System.out.println("isDir");
        }
}
```

### Source Roots ###
```
VirtualFile[] source = ProjectRootManager.getInstance(project).getContentSourceRoots();
```

### Project Modules ###
```
Module[] allModules = ModuleManager.getInstance(project).getModules();
```

### Module of PsiElement ###
```
presenterConfigModel.setModule(module);
```

### FileType ###
```
FileType fileType = StdFileTypes.JAVA;
```

### Format Class ###
```
ApplicationManager.getApplication().runWriteAction(new Runnable() {
    @Override
    public void run() {
        nameTokensPsiClass.add(newField);
        nameTokensPsiClass.add(newMethod);

        CodeStyleManager.getInstance(project).reformat(nameTokensPsiClass);
    }
});
```