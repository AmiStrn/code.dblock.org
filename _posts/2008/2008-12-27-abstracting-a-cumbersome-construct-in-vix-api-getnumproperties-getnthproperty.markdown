---
layout: post
title: "Abstracting a cumbersome construct in VIX API: GetNumProperties + GetNthProperty"
redirect_from: "/abstracting-a-cumbersome-construct-in-vix-api-getnumproperties-getnthproperty/"
date: 2008-12-27 22:15:00
tags: [vmware]
comments: true
dblog_post_id: 28
---
Did I tell you how much I love **[yield return](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/yield)**?

One of the peculiar VIX COM API constructs is the combination that returns arrays of properties. This is done with two functions: `GetNumProperties` and `GetNthProperties`. The first returns the number of property arrays returned by the job and the second fetches a property array at a given index. The first obvious step is to wrap the functions within the job class.

{% highlight c# %}
public T GetNthProperties<T>(int index, object[] properties) {
  object result = null;
  VMWareInterop.Check(_handle.GetNthProperties(index, properties, ref result));
  return (T) result;
}

public int GetNumProperties(int property) {
  return _handle.GetNumProperties(property);
}
{% endhighlight %}

We can now write such properties as `RunningVirtualMachines`.

{% highlight c# %}
public IEnumerable<VMWareVirtualMachine> RunningVirtualMachines {
   get
   {
      VMWareJob job = new VMWareJob(_handle.FindItems(
         Constants.VIX_FIND_RUNNING_VMS, null, -1, null));
      job.Wait(VMWareInterop.Timeouts.FindItemsTimeout);
      object[] properties = { Constants.VIX_PROPERTY_FOUND_ITEM_LOCATION };
      for (int i = 0; i < job.GetNumProperties((int) properties[0]); i++)
      {
         yield return this.Open(
            (string) job.GetNthProperties<object[]>(
               i, properties)[0]);
      }
   }
}
{% endhighlight %}

This is still not good enough. Let's combine the number of results and the results themselves in a `YieldWait` method.

{% highlight c# %}
public IEnumerable<object[]> YieldWait(object[] properties, int timeoutInSeconds) {
   Wait(timeoutInSeconds);
   for (int i = 0; i < GetNumProperties((int)properties[0]); i++)
   {
      yield return GetNthProperties<object[]>(i, properties);
   }
}
{% endhighlight %}

This results in a nice improvement over the previous implementation: we're interating over a resultset rather than calling methods for how many results are available and to fetch each result.

{% highlight c# %}
public IEnumerable<VMWareVirtualMachine> RunningVirtualMachines {
   get
   {
      VMWareJob job = new VMWareJob(_handle.FindItems(
         Constants.VIX_FIND_RUNNING_VMS, null, -1, null));
      object[] properties = { Constants.VIX_PROPERTY_FOUND_ITEM_LOCATION };
      foreach (object[] runningVirtualMachine in job.YieldWait(
         properties, VMWareInterop.Timeouts.FindItemsTimeout))
      {
         yield return this.Open((string) runningVirtualMachine[0]);
      }
   }
}
{% endhighlight %}
