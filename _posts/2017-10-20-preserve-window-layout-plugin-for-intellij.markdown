---
layout: post
title:  "Developing Preserve Layout Plugin for IntelliJ IDEA"
date:   2017-10-20 13:29:01 +0200
categories: dev idea jetbrains plugin java
excerpt_separator: <!--more-->
---
In this blog post we'll take a look at the development of a plugin for
IntelliJ Platform that allows us to export & import project's window layout.

![Plugin View](https://image.ibb.co/mquG36/plugin_window.jpg)

I wrote this post to address the issues I encountered during development. As of
current date (Oct 20, 2017), IntelliJ SDK documentation is far from being
complete. Below is my collection of findings regarding missing pieces.

<!--more-->

## Initial Setup
For initial setup, if you never developed a JetBrains' plugin, I highly
recommend to go through the [**Getting Started**][sdk-getting-started] section
of IntelliJ Platform SDK.

You will notice some sections are missing and greyed out. But by taking
initial setup steps you will get just enough to write your plugin. It is
important to mention that in order to **debug** a plugin you will need
the open source version of IntelliJ IDEA - IntelliJ Community Edition -
checked out and attached to the project.

## Prototyping
*If you are familiar with plugin development for IntelliJ it is safe to skip
this section.*

First thing you might look in to is the `plugin.xml` file that contains the
basic configuration of your plugin.

After filling in information fields like `<vendor>` or `<version>` we can jump
straight to the `<actions>` and add nodes for out action. Let's position our
action in the project view popup menu, the one you that appears when you
right-click on the project in IDEA.

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

Obviously, we will need a class responsible for this action. Let's create some
super simple action that will print project's name to the console. You can
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

After we run this code IDEA will spawn a new instance of itself with the
plugin installed by default. After we right click on the project (you will need
a project to be created or open) and invoke context menu. We will see that
*LayoutExporter* action appeared in this menu. If we click, we'll see a project
name in the console output.

Of course, we can assign actions to a different menu. For example, in
`preserve-layout-plugin` actions are assigned to `WindowMenu`.

## Window layout in IDEA
Stepping into the actual logic we might ask ourselves where is the layout of
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

Possessing the manager we can now ask it for a current layout. An instance of
`DesktopLayout` will be returned. Then we will ask `DesktopLayout` to return
`org.jdom.Element` representation of itself.
{% highlight java %}
DesktopLayout dl = mgr.getLayout();
Element layout = dl.writeExternal("layout");
{% endhighlight %}


## File Saver dialog & Saving/writing a file
**Save File dialog:**
Somehow, file saving dialog in IntelliJ is invoked in a different way,
comparing to File Chooser. It is also not mentioned yet in documentation.
However, you can find a `FileSaverDescriptor` class that when passed to
`FileChooserFactory` as a parameter will produce a correct dialog.

**Writing File in IntelliJ:**
Actual data writing is simple but not obvious. It is required to execute disk
write operations outside of the main thread. To address this IntelliJ provides
a very convenient `WriteCommandAction` tool.

Besides that, best approach to write a file to disk is thrugh the `VirtualFile`
abstraction layer.

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
able to invoke native file chooser dialogs on every platform.

{% highlight java %}
FileChooser.chooseFile(
        FileChooserDescriptorFactory.createSingleFileDescriptor(),
        e.getProject(),
        null,
        file -> importLayoutFileToProject(file.getCanonicalPath(), e.getProject()));
{% endhighlight %}

Let's implement the callback function.
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

`importLayoutFileToProject` calls 2 helping methods to parse and apply the
layout to IDE.
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

Having all that we can now import and export the window layout. However, it
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

Here we can change the icon
{% highlight java %}
Notifications.Bus.notify(new Notification(
        "Preserve Layout",
        "Export Failed",
        ex.getMessage(),
        NotificationType.ERROR
), e.getProject());
{% endhighlight %}

## Support every JetBrains IDE
Last thing but not the least important. To support every JetBrains IDE it is
required to specify the `<depends>` tag in `<plugin>`. More details about
plugin compatibility are available at [SDK docs][sdk-plugin-compativility].
{% highlight xml %}
<depends>com.intellij.modules.lang</depends>
{% endhighlight %}

[plugin-gh]:    (https://github.com/vduseev/preserve-layout-plugin)
[plugin-page]:  (https://plugins.jetbrains.com/plugin/10097-preserve-layout-plugin)
[sdk-getting-started]: (https://www.jetbrains.org/intellij/sdk/docs/basics/getting_started.html)
[sdk-about-projects]: (https://www.jetbrains.com/help/idea/about-projects.html)
[sdk-abandon-old]: (https://intellij-support.jetbrains.com/hc/en-us/community/posts/206167769/comments/206288985)
[sdk-plugin-compativility]: (ttp://www.jetbrains.org/intellij/sdk/docs/basics/getting_started/plugin_compatibility.html)
