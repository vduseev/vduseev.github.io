---
title: "Writing an IntelliJ IDEA plugin to manage window layouts"
date: 2017-10-20 13:29:01 +0200
keywords: intellij idea jetbrains plugin java ToolWindowManagerImpl DesktopLayout settings export import
description: "How to create an IntelliJ plugin with window layout management, menu, notifications, and import/export."
image: https://image.ibb.co/mquG36/plugin_window.jpg
redirect_from: "/preserve-window-layout-plugin-for-intellij/"

---
There are many IntelliJ plugins out there. Still, existing SDK documentation
does not cover every development step. Here I describe how a basic IntelliJ
plugin with menu and import/export functionality can be developed.

![IntelliJ IDEA plugin to manage window layouts](https://image.ibb.co/mquG36/plugin_window.jpg)

<!--more-->

## Plugin's purpose
Sometimes, after working on some project for a long enough period, you end up with a
particular window layout that is convenient for you, e.g., console on the right side,
source code on the left, debug window at the top.

However, if you open the same project on a different machine, the layout you've so delicately configured will be lost. Same situation will happen if you want to reuse a layout from some other project.

**Preserve Layout Plugin** allows you to export the layout of any
IntelliJ project and then import it back. Export is done using the XML format.

## About this blog post
I wrote this post to address the issues I encountered during development of
such plugin. As of
current date (Oct 20, 2017), IntelliJ SDK documentation is way far from being
complete. Below is my collection of findings regarding missing pieces.

## Initial Setup
For initial setup, if you never developed a JetBrains' plugin, I highly
recommend to go through the [**Getting Started**][sdk-getting-started] section
of IntelliJ Platform SDK.

You will notice that some sections are missing and greyed out. However, just
by following these
initial setup steps you will get just enough to create your own plugin. It is
important to mention that in order to **debug** a plugin you will need
the source code of IntelliJ Community Edition checked out and attached to the
project.

## Prototyping
*If you are familiar with plugin development for IntelliJ, it is safe to skip
this section.*

To play around a little bit, let's create a very simple plugin with one action.
Just to test the things and confirm that setup is working.

First thing you might look in to is the `plugin.xml` file that contains the
basic configuration of your plugin.

After filling in the standard information fields like `<vendor>` or `<version>`
we can jump straight to the `<actions>` section.  
Here we can statically position and initialize our actions.

`plugin.xml` is sufficient enough for most needs. However, when doing complex
stuff, we can initialize actions dynamically in Java.

Let's position our
action in the Project View popup menu, the one that appears when you
right-click the project name in IDEA.

**Information tags:**
{% highlight xml %}
<id>com.duseev.intellij.preservelayout</id>
<name>Preserve Layout Plugin</name>
<version>1.0</version>
<vendor email="vagiz@duseev.com" url="http://duseev.com">Duseev.com</vendor>
{% endhighlight %}

**Decribing actions:**
{% highlight xml %}
<actions>
  <action id="ExportLayout"
          class="com.duseev.intellij.preservelayout.LayoutExporter"
          text="LayoutExporter"
          description="LayoutExporter">
  <add-to-group group-id="ProjectViewPopupMenu"
                anchor="after"
                relative-to-action="ExternalToolsGroup"/>
  </action>
</actions>
{% endhighlight %}

You can see that we created a unique `id` for this action. A class for the
should also be specified. A name and description will be `LayoutExporter`. The
`<add-to-group ...>` tag is the one responsible for tying our action to the
specific menu in IDEA.

Obviously, we will need a class responsible for this action. Let's make it
super simple: it will print project's name to the console. You can
notice that we inherit from `AnAction` class and override the
`actionPerformed` method.
{% highlight java %}
package com.duseev.intellij.preservelayout;

import com.intellij.openapi.actionSystem.AnAction;
import com.intellij.openapi.actionSystem.AnActionEvent;
import com.intellij.openapi.project.*;

public class LayoutExporter extends AnAction {

   @Override
   public void actionPerformed(AnActionEvent e) {
       // TODO: insert action logic here
       Project project = e.getProject();
       System.out.println(project.getName());
   }
}
{% endhighlight %}

After we run this code, IDEA will spawn a new instance of itself with our
plugin installed by default. After we right click on the project (you will need
a project to be created or open) and invoke context menu, we will see that
*LayoutExporter* action appeared in this menu. Clicking this action will
print a project name to the console output.

Of course, we can assign actions to a different menu. For example, in
`preserve-layout-plugin` actions are added to the `WindowMenu`.

## Window layout in IDEA
Stepping into the actual logic, we might ask ourselves where is the layout of
current project stored in IntelliJ IDEA? Turns out it can be stored in [two ways][sdk-about-projects]:
* `directory based project`: in `workspace.xml` file under `.idea` directory
* `file based project`: in `.iws` file in a project directory

JetBrains is planning to [deprecate][sdk-abandon-old] the file based format.
But we still have to support both options.

**Getting current window layout:**
To obtain current layout state we obtain an instance of the
`ToolWindowManagerImpl`:
{% highlight java %}
ToolWindowManagerImpl mgr = (ToolWindowManagerImpl) ToolWindowManager.getInstance(e.getProject());
{% endhighlight %}

Having the manager, we can now ask it for a current layout. An instance of
`DesktopLayout` will be returned. Then we cas use `DesktopLayout` to return
`org.jdom.Element` representation of itself.
{% highlight java %}
DesktopLayout dl = mgr.getLayout();
Element layout = dl.writeExternal("layout");
{% endhighlight %}

Next step is to save this XML/DOM Element somewhere.

## File Saver dialog & Saving/writing a file
**File Saving dialog:**
Somehow, file saving dialog in IntelliJ is invoked in a different way,
comparing to File Chooser. It is also not mentioned yet in documentation.
However, you can come across a `FileSaverDescriptor` class, which when
instantiated and passed to
`FileChooserFactory` as a parameter will produce a correct dialog window.

**Writing to a File in IntelliJ:**
Actual data writing is simple, but not obvious. It is required to execute
"write" operations outside of the main thread. To address this,
IntelliJ provides
a very convenient `WriteCommandAction` abstraction.

Besides that, best approach to write a file to a disk is through
the `VirtualFile` abstraction layer.

Let's take a look at a composed example of logic required to save an XML to
a file in IntelliJ.

{% highlight java %}
// Use XML to covert Element to String
XMLOutputter outputter = new XMLOutputter(Format.getPrettyFormat());
String exportContent = outputter.outputString(doc);

// Describe file saving dialog
FileSaverDescriptor descriptor = new FileSaverDescriptor(
        "Save Layout",
        "Choose path for layout export...",
        "xml");

// Obtain a file wrapper from FileChooserFactory
VirtualFileWrapper wrapper = FileChooserFactory
        .getInstance()
        .createSaveFileDialog(descriptor, e.getProject())
        .save(e.getProject().getBaseDir(), "layout.xml");

if (wrapper == null) return;
VirtualFile file = wrapper.getVirtualFile(true);

if (file == null) throw new Exception("Couldn't create new file");

// Write to disk outside of main GUI thread
new WriteCommandAction.Simple(e.getProject()) {
    @Override
    protected void run() throws Throwable {
        VfsUtil.saveText(file, exportContent);
    }
}.execute();
{% endhighlight %}

## File Chooser dialog & importing layout
The best way to select a file in IntelliJ is to use the `FileChooser` dialog
and the overloaded function `chooseFile` that accepts a callback parameter to
be called after the file is selected. When using this approach IntelliJ is
able to invoke native file chooser dialog window on every platform.

{% highlight java %}
FileChooser.chooseFile(
    FileChooserDescriptorFactory.createSingleFileDescriptor(),
    e.getProject(),
    null,
    file -> importLayoutFileToProject(file.getCanonicalPath(), e.getProject())
);
{% endhighlight %}

Now we need a callback function `importLayoutFileToProject()`.
{% highlight java %}
private void importLayoutFileToProject(String path, Project project) {
    if (path == null) return;
    if (project == null) return;

    try {
        Element layout = parseLayoutFile(path);
        applyLayoutToProject(layout, project);
    } catch (Exception ex) {
        // notify user
    }
}
{% endhighlight %}

As you can notice,
`importLayoutFileToProject` makes 2 calls to the helping methods.
One to parse the imported XML file, and the other to apply the parsed
layout to IDE. Let's implement them as well.
{% highlight java %}
private Element parseLayoutFile(String path) throws Exception {
    Document doc = new SAXBuilder().build(new File(path));
    return doc.getRootElement();
}

private void applyLayoutToProject(Element layout, Project project) {
    ToolWindowManagerImpl mgr = (ToolWindowManagerImpl) ToolWindowManager.getInstance(project);
    DesktopLayout dl = mgr.getLayout();
    dl.readExternal(layout);
}
{% endhighlight %}

Having all that we can now import and export window layout's of any project.
However, it
would be nice to notify a user about results of the export/import process.

## IntelliJ Notifications
Basic notifications are very easy to fire in IntelliJ:
{% highlight java %}
Notifications.Bus.notify(new Notification(
    "Preserve Layout",
    "Successful Export",
    "Saved to " + file.getCanonicalPath(),
    NotificationType.INFORMATION
), e.getProject());
{% endhighlight %}

Here we can change the icon to emphasize the `ERROR` status.
{% highlight java %}
Notifications.Bus.notify(new Notification(
    "Preserve Layout",
    "Export Failed",
    ex.getMessage(),
    NotificationType.ERROR
), e.getProject());
{% endhighlight %}

## Make plugin runnable in every JetBrains IDE
Last thing but not the least important. To support every JetBrains IDE it is
required to specify the `<depends>` tag in `<plugin>`. More details about
plugin compatibility are available at [SDK docs][sdk-plugin-compativility].
{% highlight xml %}
<depends>com.intellij.modules.lang</depends>
{% endhighlight %}

### Special thanks
This plugin has been developed thanks to [Alexander Zolotov][azolotov-github] from JetBrains team.
He provided tons of invaluable tips and advices during the development of this plugin.


[plugin-gh]:    https://github.com/vduseev/preserve-layout-plugin
[plugin-page]:  https://plugins.jetbrains.com/plugin/10097-preserve-layout-plugin
[sdk-getting-started]: https://www.jetbrains.org/intellij/sdk/docs/basics/getting_started.html
[sdk-about-projects]: https://www.jetbrains.com/help/idea/about-projects.html
[sdk-abandon-old]: https://intellij-support.jetbrains.com/hc/en-us/community/posts/206167769/comments/206288985
[sdk-plugin-compativility]: http://www.jetbrains.org/intellij/sdk/docs/basics/getting_started/plugin_compatibility.html
[azolotov-github]: https://github.com/zolotov
