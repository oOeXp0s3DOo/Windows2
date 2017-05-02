<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <!-- This inline task executes c# code. -->
  <!-- C:\Windows\Microsoft.NET\Framework64\v4.0.30319\msbuild.exe pshell.xml -->
   <!-- Author: Casey Smith, Twitter: @subTee -->
  <!-- License: BSD 3-Clause -->
  <Target Name="Hello">
   <FragmentExample />
   <ClassExample />
  </Target>
  <UsingTask
    TaskName="FragmentExample"
    TaskFactory="CodeTaskFactory"
    AssemblyFile="C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Build.Tasks.v4.0.dll" >
    <ParameterGroup/>
    <Task>
      <Using Namespace="System" />
	  <Using Namespace="System.IO" />
      <Code Type="Fragment" Language="cs">
        <![CDATA[
			    Console.WriteLine("Hello From Fragment");
        ]]>
      </Code>
    </Task>
	</UsingTask>
	<UsingTask
    TaskName="ClassExample"
    TaskFactory="CodeTaskFactory"
    AssemblyFile="C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Build.Tasks.v4.0.dll" >
	<Task>
	  <Reference Include="System.Management.Automation" />
      <Code Type="Class" Language="cs">
        <![CDATA[
		
			using System;
			using System.IO;
			using System.Diagnostics;
			using System.Reflection;
			using System.Runtime.InteropServices;

			//Add For PowerShell Invocation
			using System.Collections.ObjectModel;
			using System.Management.Automation;
			using System.Management.Automation.Runspaces;
			using System.Text;


			using Microsoft.Build.Framework;
			using Microsoft.Build.Utilities;
							
			public class ClassExample :  Task, ITask
			{
				public override bool Execute()
				{
					
					while(true)
					{
						
						Console.Write("PS >");
						string x = Console.ReadLine();
						try
						{
							Console.WriteLine(RunPSCommand(x));
						}
						catch (Exception e)
						{
							Console.WriteLine(e.Message);
						}
					}
					
								return true;
				}
				
				//Based on Jared Atkinson's And Justin Warner's Work
				public static string RunPSCommand(string cmd)
				{
					//Init stuff
					Runspace runspace = RunspaceFactory.CreateRunspace();
					runspace.Open();
					RunspaceInvoke scriptInvoker = new RunspaceInvoke(runspace);
					Pipeline pipeline = runspace.CreatePipeline();

					//Add commands
					pipeline.Commands.AddScript(cmd);

					//Prep PS for string output and invoke
					pipeline.Commands.Add("Out-String");
					Collection<PSObject> results = pipeline.Invoke();
					runspace.Close();

					//Convert records to strings
					StringBuilder stringBuilder = new StringBuilder();
					foreach (PSObject obj in results)
					{
						stringBuilder.Append(obj);
					}
					return stringBuilder.ToString().Trim();
				 }
				 
				 public static void RunPSFile(string script)
				{
					PowerShell ps = PowerShell.Create();
					ps.AddScript(script).Invoke();
				}
				
				
			}
			
			
 
			
        ]]>
      </Code>
    </Task>
  </UsingTask>
</Project>
