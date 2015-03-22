#This page illustrates the procedure of building XMind application using source code.

# What you need #

  * XMind source package: http://code.google.com/p/xmind3/downloads/
  * Eclipse IDE: http://www.eclipse.org/downloads/
  * XMind standalone application: http://www.xmind.net/downloads/

# Instructions #

## Step 1 - Installation ##

  1. Install Eclipse IDE to folder {ECLIPSE}
  1. Install XMind standalone application to folder {XMIND}
  1. Copy the following jar files from {XMIND}/plugins/ (on OS X, right-click or control-click on XMind in Applications folder, select 'Show Package Contents', select Contents -> Resources -> plugins) to {ECLIPSE}/plugins/
    * net.sourceforge.jazzy\_0.5.0.jar
    * org.bouncycastle\_1.3.8.jar
    * org.json\_1.0.0.jar
  1. Launch Eclipse IDE and create a new workspace {WORKSPACE}
  1. Select Help menu -> Software Updates... -> Available Software, install:
    * 'Graphical Editing Framework Draw2d'
    * (recommended) 'Graphical Editing Framework Draw2d Developer Resource'

## Step 2 - Importation ##

  1. In Eclipse, select File menu -> Import... -> Existing Projects into Workspace
  1. Select the archive file (source package) downloaded from this site
  1. Select all projects and click Finish
  1. After the importing process is over, restart Eclipse

## Step 3 - Building ##

  1. Select org.xmind.cathy.win32:
    * If not working on Windows, delete this project
  1. If there's still building errors, try Project menu -> Clean... -> Clean all projects

## Step 4 - Launching ##

  1. Select Run menu -> Run Configurations... (same for Debug Configurations)
  1. Right-click (control-click on OS X) on Eclipse Application -> New
  1. In Main tab,
    * Name: XMind (or any)
    * Location: ${workspace\_loc}/../runtime-cathy (or any)
    * Run a product: org.xmind.cathy.product
  1. Switch to Plug-ins tab,
    1. Launch with: plug-ins selected below only
    1. Deselect All
    1. Check only those XMind-related plug-ins, language pack fragments (xx.xx.xx.nl\_xx) may be included as needed
    1. Add Required Plug-ins
  1. Apply changes and Run!

# References #
  * Sincere thanks to Rodrigo Garcia and Max Larin for sharing their experiences in [building XMind from source](http://groups.google.com/group/xmind-dev/browse_thread/thread/ee6fcf7d3bb0f825)
  * [Basis of Eclipse Rich Client Platform (RCP)](http://www.eclipse.org/articles/Article-RCP-1/tutorial1.html)
  * [Eclipse Platform Plug-in Developer Guide](http://help.eclipse.org/stable/index.jsp?nav=/2)