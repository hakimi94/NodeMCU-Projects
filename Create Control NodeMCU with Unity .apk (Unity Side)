using UnityEngine;
using UnityEngine.UI;
using System.Collections;
using System;
using System.IO;
using System.Net;
using System.Net.Sockets;

public class StringControlOverNetwork : MonoBehaviour
{

    public GameObject ConnectionSetupPanel;
    public InputField IP_Text;
    public InputField IPort_Text;
    public Text IPofThis;

    public GameObject ControlPanelPanel;
    public GameObject LoginPanel;
    public InputField UserName_Text;
    public InputField Pass_Text;

    public GameObject CommandsPanel;
    public InputField Command_Text;

    public Text RecievdDataTxt;
    public Text Log;

    public string m_IPAdress;
    public int kPort;
    string Local_IP;
    string RecivedString;
    bool LogInState;
    internal Boolean socketReady = false;

    TcpClient mySocket;
    NetworkStream theStream;
    StreamWriter theWriter;
    StreamReader theReader;
    //String Host = "localhost";
    //Int32 Port = 5333;

    void Start()
    {
        IP_Text.text = "192.168.1.150";
        IPort_Text.text = "1234";
        LogInState = false;
        ControlPanelPanel.GetComponent<GUI_State_with_Color>().GotoState(1);
        ConnectionSetupPanel.GetComponent<GUI_State_with_Color>().GotoState(0);
        LoginPanel.GetComponent<GUI_State_with_Color>().GotoState(1);
        CommandsPanel.GetComponent<GUI_State_with_Color>().GotoState(1);
    }
    void Update()
    {
        string receivedText = readSocket();
        if (receivedText != "")
        {
            RecivedString = receivedText;
            RecievdDataTxt.text += receivedText;
            if (receivedText == "ID is correct") { LogInState = true; }
        }
        if (socketReady == true)
        {
            IPofThis.text = "Connected at: " + LocalIPAddress();
            ConnectionSetupPanel.GetComponent<GUI_State_with_Color>().GotoState(1);
            ControlPanelPanel.GetComponent<GUI_State_with_Color>().GotoState(0);
            if (LogInState == true)
            {
                LoginPanel.GetComponent<GUI_State_with_Color>().GotoState(1);
                CommandsPanel.GetComponent<GUI_State_with_Color>().GotoState(0);
            }
            else
            {
                LoginPanel.GetComponent<GUI_State_with_Color>().GotoState(0);
                CommandsPanel.GetComponent<GUI_State_with_Color>().GotoState(1);
            }
        }
        else
        {
            ConnectionSetupPanel.GetComponent<GUI_State_with_Color>().GotoState(0);
            ControlPanelPanel.GetComponent<GUI_State_with_Color>().GotoState(1);
        }
    }
    void OnGUI()
    {
        /*if (!socketReady)
        {
            if (GUILayout.Button("Connect"))
            {
                setupSocket();
                writeSocket("Command = User&Pass >> User:Pass ");
                Local_IP = LocalIPAddress();
            }
        }
        if (GUILayout.Button("Turn LED Green On"))
        {
            writeSocket("Command = GreenLED >> On ");
        }
        if (GUILayout.Button("Turn LED Red On"))
        {
            writeSocket("Command = RedLED >> On ");
        }
        if (GUILayout.Button("Close"))
        {
            writeSocket("Command = Close The Connection ");
            closeSocket();
        }
        GUI.Label(new Rect(0, 40, 1000, 400), RecivedString);
        GUI.Label(new Rect(0, 200, 1000, 400), Local_IP);

        if (socketReady)
        {
            
        }*/
    }
    public void Connect()
    {
        m_IPAdress = IP_Text.text;
        kPort = int.Parse(IPort_Text.text);
        Disconnect();
        setupSocket();
    }
    public void LogIn()
    {
        setupSocket();
        string ID = "Command = User&Pass >> " + UserName_Text.text + ":" + Pass_Text.text;
        writeSocket(ID);
    }
    public void SendData()
    {
        writeSocket(Command_Text.text);
    }
    public void Disconnect()
    {
        writeSocket("Command = Close The Connection");
        ConnectionSetupPanel.GetComponent<GUI_State_with_Color>().GotoState(0);
        ControlPanelPanel.GetComponent<GUI_State_with_Color>().GotoState(1);
        LogInState = false;
        closeSocket();
    }
    public void setupSocket()
    {
        try
        {
            //mySocket = new TcpClient(Host, Port);
            mySocket = new TcpClient(AddressFamily.InterNetwork);
            System.Net.IPAddress remoteIPAddress = System.Net.IPAddress.Parse(m_IPAdress);
            System.Net.IPEndPoint remoteEndPoint = new System.Net.IPEndPoint(remoteIPAddress, kPort);
            mySocket.Connect(remoteEndPoint);

            theStream = mySocket.GetStream();
            theWriter = new StreamWriter(theStream);
            theReader = new StreamReader(theStream);
            socketReady = true;

            Log.text += "Connected to :" + IP_Text.text + "\n";
        }
        catch (Exception e)
        {
            Debug.Log("Socket error: " + e);
            Log.text += "Socket error: " + e + "\n";
            socketReady = false;
        }
    }
    public void writeSocket(string theLine)
    {
        if (!socketReady)
            return;

        String foo = theLine + "\r\n";
        theWriter.Write(foo);
        theWriter.Flush();
    }
    public String readSocket()
    {
        if (!socketReady)
            return "";

        if (theStream.DataAvailable)
            return theReader.ReadLine();
        //return theReader.ReadToEnd();
        return "";

    }
    public void closeSocket()
    {
        if (!socketReady)
            return;

        theWriter.Close();
        theReader.Close();
        mySocket.Close();
        socketReady = false;
        Log.text += "Disconnected from :" + IPort_Text.text + "\n";
    }
    void OnApplicationQuit()
    {
        if (!socketReady)
            return;

        theWriter.Close();
        theReader.Close();
        mySocket.Close();
        socketReady = false;
        Log.text += "Disconnected from :" + IPort_Text.text + "\n";
    }
    void OnApplicationPause()
    {
        if (!socketReady)
            return;

        theWriter.Close();
        theReader.Close();
        mySocket.Close();
        socketReady = false;
        Log.text += "Disconnected from :" + IPort_Text.text + "\n";
    }
    public string LocalIPAddress()
    {
        IPHostEntry host;
        string localIP = "";
        host = Dns.GetHostEntry(Dns.GetHostName());
        foreach (IPAddress ip in host.AddressList)
        {
            if (ip.AddressFamily == AddressFamily.InterNetwork)
            {
                localIP = ip.ToString();
                break;
            }
        }
        return localIP;
    }

    // GUI States ________________________________________________

    public void SetConnectionPanel_Appear()
    { 
        ConnectionSetupPanel.GetComponent<GUI_State_with_Color>().GotoState(1);
    }
    public void SetConnectionPanel_Hide()
    {
        ConnectionSetupPanel.GetComponent<GUI_State_with_Color>().GotoState(0);
    }
    public void LogInPanel_Appear()
    {
        LoginPanel.GetComponent<GUI_State_with_Color>().GotoState(0);
    }
    public void LogInPanel_Hide()
    {
        LoginPanel.GetComponent<GUI_State_with_Color>().GotoState(1);
    }
    public void CommandsPanel_Appear()
    {
        CommandsPanel.GetComponent<GUI_State_with_Color>().GotoState(0);
    }
    public void CommandsPanel_Hide()
    {
        CommandsPanel.GetComponent<GUI_State_with_Color>().GotoState(1);
    }
}
