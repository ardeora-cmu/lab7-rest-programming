# lab7-rest-programming

# 95-702 Distributed Systems             REST Programming

## Modifying a working client and server handling POST, PUT, GET, and DELETE

The first task in this lab is to get the following code running in IntelliJ. Create a standard
Java project named WebServiceDesignStyles3ClientSideProject. Within that project, use the client side code
provided. Create a standard Java Web Application project with the server side code provided. Name
the server side project WebServiceDesignStyles3ServerSideProject.

Deploy the server side project to TomEE. Run the client side code. Spend some time studying
both the client and the server.

web.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>TestServlet</servlet-name>
        <servlet-class>edu.cmu.andrew.mm6.TestServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>TestServlet</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
</web-app>

```

URL:  
http://localhost:8080/ServerSideProject

Application Context:  
/ServerSideProject

:checkered_flag:**Completion of Task 1 is this lab's checkpoint.**


The second task is to modify the client so that it provides methods called getVariableList() and
the lower level method named doGetList(). Note the call to the getVariableList method (commented out)
within the client side code. After modifying the server, remove the comment symbols from this
call. That is, the method should actually work.

The getVariableList method has the following signature and description:

```
public static String getVariableList()
// makes a call to doGetList()
// returns a list of all variable defined on the server.

```

The doGetList method has the following signature and description:

```
public static int doGetList(Result r)
// Makes an HTTP GET request to the server. This is similar to the doGet provided on the client
// but this one uses a different URL.
// This method makes a call to the HTTP GET method using
// http://localhost:8080/WebServiceDesignStyles3ProjectServerLab/VariableMemory/"

```

The third task is to modify the doGet method on the server. Currently it returns
an HTTP 401 if a name is not provided in the URL. Your new doGet method will return
a list of variable names found in the map. This list of variable names will be returned
to the client for display. Clients will now be able to use two different URL's with a GET
request. One will be used to return the value of a variable and the other will be able
display the list of variables found within the map on the server. You are not being asked
to return the values that are also stored in the map.

:checkered_flag:**Show your TA your working solution. My solution has the following output on the client
side:**

```
Begin main of REST lab.
Assign 100 to the variable named x
Assign 199 to the variable named x
Sending a GET request for x
199
Sending a DELETE request for x
x is deleted but let's try to read it
Error from server 401
It is
a wonderful day
in the neighborhood!
abc
End main of REST lab
```


Client side code

```

// ****************  Client side code  *********************
package edu.cmu.andrew.mm6;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.DataOutputStream;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;

// A simple class to wrap a result.
class Result {
    String value;

    public String getValue() {
        return value;
    }
    public void setValue(String value) {
        this.value = value;
    }
}
public class Main {


    public static void main(String[] args) throws Exception{

        System.out.println("Begin main of REST lab.");
        System.out.println("Assign 100 to the variable named x");
        assign("x","100");
        System.out.println("Assign 199 to the variable named x");
        assign("x","199");
        System.out.println("Sending a GET request for x");
        // Get the value associated with a name on the server
        System.out.println(read("x"));
        System.out.println("Sending a DELETE request for x");
        clear("x");
        System.out.println("x is deleted but let's try to read it");
        System.out.println(read("x"));

        assign("a","It is\n ");
        assign("b","a wonderful day\n");
        assign("c","in the neighborhood!\n");

        System.out.println(read("a"));
        System.out.println(read("b"));
        System.out.println(read("c"));

        //System.out.println(getVariableList());

        System.out.println("End main of REST lab");
    }

    // assign a string value to a string name (One character names for demo)
    public static boolean assign(String name, String value) {
        // We always want to be able to assign so we may need to PUT or POST.
        // Try to PUT, if that fails then try to POST
        if(doPut(name,value) == 200) {
            return true;
        }
        else {
            if(doPost(name,value) == 200) {
                return true;
            }
        }
        return false;
    }

    // read a value associated with a name from the server
    // return either the value read or an error message
    public static String read(String name) {
        Result r = new Result();
        int status = 0;
        if((status = doGet(name,r)) != 200) return "Error from server "+ status;
        return r.getValue();
    }
    // delete a variable on the server
    // if the server sends an error return false to the caller
    public static boolean clear(String name) {
        if(doDelete(name) == 200) return true;
        else return false;
    }

    // Low level routine to make an HTTP POST request
    // Note, POST does not use the URL line for its message to the server
    public static int doPost(String name, String value) {

        int status = 0;
        String output;

        try {
                // Make call to a particular URL
		URL url = new URL("http://localhost:8080/WebServiceDesignStyles3ProjectServerLab/VariableMemory/");
		HttpURLConnection conn = (HttpURLConnection) url.openConnection();

                // set request method to POST and send name value pair
                conn.setRequestMethod("POST");
		conn.setDoOutput(true);
                // write to POST data area
                OutputStreamWriter out = new OutputStreamWriter(conn.getOutputStream());
                out.write(name + "=" + value);
                out.close();

                // get HTTP response code sent by server
                status = conn.getResponseCode();

                //close the connection
		conn.disconnect();
	    }
            // handle exceptions
            catch (MalformedURLException e) {
		      e.printStackTrace();
            }
            catch (IOException e) {
		      e.printStackTrace();
	    }

            // return HTTP status

            return status;
    }

     public static int doGet(String name, Result r) {



         // Make an HTTP GET passing the name on the URL line

         r.setValue("");
         String response = "";
         HttpURLConnection conn;
         int status = 0;

         try {

                // pass the name on the URL line
		URL url = new URL("http://localhost:8080/WebServiceDesignStyles3ProjectServerLab/VariableMemory" + "//"+name);
		conn = (HttpURLConnection) url.openConnection();
		conn.setRequestMethod("GET");
                // tell the server what format we want back
		conn.setRequestProperty("Accept", "text/plain");

                // wait for response
                status = conn.getResponseCode();

                // If things went poorly, don't try to read any response, just return.
		if (status != 200) {
                    // not using msg
                    String msg = conn.getResponseMessage();
                    return conn.getResponseCode();
                }
                String output = "";
                // things went well so let's read the response
                BufferedReader br = new BufferedReader(new InputStreamReader(
			(conn.getInputStream())));

		while ((output = br.readLine()) != null) {
			response += output;

		}

		conn.disconnect();

	    }
                catch (MalformedURLException e) {
		   e.printStackTrace();
	    }   catch (IOException e) {
		   e.printStackTrace();
	    }

         // return value from server
         // set the response object
         r.setValue(response);
         // return HTTP status to caller
         return status;
    }

    // Low level routine to make an HTTP PUT request
    // Note, PUT does not use the URL line for its message to the server
    public static int doPut(String name, String value) {


         int status = 0;
         try {
		URL url = new URL("http://localhost:8080/WebServiceDesignStyles3ProjectServerLab/VariableMemory/");
		HttpURLConnection conn = (HttpURLConnection) url.openConnection();
		conn.setRequestMethod("PUT");
		conn.setDoOutput(true);
                OutputStreamWriter out = new OutputStreamWriter(conn.getOutputStream());
                out.write(name + "=" + value);
                out.close();
		status = conn.getResponseCode();

                conn.disconnect();

	     }
             catch (MalformedURLException e) {
		   e.printStackTrace();
	     } catch (IOException e) {
		   e.printStackTrace();
	     }
        return status;
    }

    // Send an HTTP DELETE to server along with name on the URL line
    // We need not read the response, we are only interested in the HTTP status
    // code.
    public static int doDelete(String name) {

        int status = 0;

         try {
		URL url = new URL("http://localhost:8080/WebServiceDesignStyles3ProjectServerLab/VariableMemory" + "//"+name);
		HttpURLConnection conn = (HttpURLConnection) url.openConnection();
		conn.setRequestMethod("DELETE");
                status = conn.getResponseCode();
		conn.disconnect();
	     }
             catch (MalformedURLException e) {
		   e.printStackTrace();
	     } catch (IOException e) {
		   e.printStackTrace();
	     }
        return status;
    }

}
```

Server side code:

```

// ******************* Server Side Code ********************
// 95-702 HTTP Lab exercise
// Working server handling POST, PUT, GET, and DELETE

// Server side code


package edu.cmu.andrew.mm6;

import java.io.IOException;
import java.io.PrintWriter;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.*;

// This example demonstrates Java servlets and HTTP
// This web service operates on string keys mapped to string values.

@WebServlet(name = "VariableMemory", urlPatterns = {"/VariableMemory/*"})
public class VariableMemory extends HttpServlet {

    // This map holds key value pairs
    private static Map memory = new TreeMap();

    // GET returns a value given a key
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        System.out.println("Console: doGET visited");

        String result = "";

        // The name is on the path /name so skip over the '/'
        String name = (request.getPathInfo()).substring(1);

        // return 401 if name not provided
        if(name.equals("")) {
            response.setStatus(401);
            return;
        }

        // Look up the name from variable memory
        String value = (String)memory.get(name);

        // return 401 if name not in map
        if(value == null || value.equals("")) {
            // no variable name found in map
            response.setStatus(401);
            return;
        }

        // Things went well so set the HTTP response code to 200 OK
        response.setStatus(200);
        // tell the client the type of the response
        response.setContentType("text/plain;charset=UTF-8");

        // return the value from a GET request
        result = value;
        PrintWriter out = response.getWriter();
        out.println(result);
    }

    // Delete an existing variable from memory. If no such variable then return a 401
    @Override
    protected void doDelete(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {


        System.out.println("Console: doDelete visited");

        String result = "";

        // The name is on the path /name so skip over the '/'
        String name = (request.getPathInfo()).substring(1);

        if(name.equals("")) {
            // no variable name return 401
            response.setStatus(401);
            return;
        }

        // Look up the name from variable memory
        String value = (String)memory.get(name);

        if(value == null || value.equals("")) {
            // no variable name found in map so return 401
            response.setStatus(401);
            return;
        }

        // delete the name
        memory.remove(name);

        // Set HTTP response code to 200 OK
        response.setStatus(200);

    }

    // POST is used to create a new variable
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        System.out.println("Console: doPost visited");

        // To look at what the client accepts examine request.getHeader("Accept")
        // We are not using the accept header here.

        // Read what the client has placed in the POST data area
        BufferedReader br = new BufferedReader(new InputStreamReader(request.getInputStream()));
        String data = br.readLine();

        // extract variable name from request data (variable names are a single character)
        String variableName = "" + data.charAt(0);

        // extract value after the equals sign

        String valString = data.substring(2);

        if(variableName.equals("") || valString.equals("")) {
            // missing input return 401
            response.setStatus(401);
            return;
        }

        String result = "";

        // If the variable is already in memory, let's return an error
        if(memory.get(variableName) != null) {
            response.setStatus(409);
            return;
        }
        else {
            // Not in memory so store name and value in the map
            memory.put(variableName, valString);

            // prepare response code
            response.setStatus(200);
            return;

        }
    }
    /* In this example, we use Put to update an existing variable.  */
    @Override
    protected void doPut(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        System.out.println("Console: doPut visited");
        // Read what the client has placed in the PUT data area
        BufferedReader br = new BufferedReader(new InputStreamReader(request.getInputStream()));
        String data = br.readLine();

        // extract variable name from request data (variable names are a single character)
        String variableName = "" + data.charAt(0);

        // extract value after the equals sign

        String valString = data.substring(2);

        if(variableName.equals("") || valString.equals("")) {
            // missing input return 401
            response.setStatus(401);
            return;
        }

        String result = "";

        // If the variable is not already in memory, let's return an error
        if(memory.get(variableName) == null) {
            response.setStatus(409);
            return;
        }
        else {
            // The name is in memory so store the new name and value in the map
            memory.put(variableName, valString);
            // prepare response code
            response.setStatus(200);
            return;

        }
    }

}
```
