{
 "cells": [
  {
   "cell_type": "code",
   "execution_count": 1,
   "id": "0c1da82c",
   "metadata": {},
   "outputs": [],
   "source": [
    "#Import libraries\n",
    "import pandas as pd\n",
    "import dash\n",
    "import dash_html_components as html\n",
    "import dash_core_components as dcc\n",
    "from dash.dependencies import Input, Output, State\n",
    "import plotly.graph_objects as go\n",
    "import plotly.express as px\n",
    "from dash import no_update"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "id": "6c799cb8",
   "metadata": {},
   "outputs": [],
   "source": [
    "#Read the airline data\n",
    "airline_data =  pd.read_csv('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DV0101EN-SkillsNetwork/Data%20Files/airline_data.csv', \n",
    "                            encoding = \"ISO-8859-1\",\n",
    "                            dtype={'Div1Airport': str, 'Div1TailNum': str, \n",
    "                                   'Div2Airport': str, 'Div2TailNum': str})"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "id": "1ba762b6",
   "metadata": {},
   "outputs": [],
   "source": [
    "#List of years\n",
    "year_list = [i for i in range(2005, 2021, 1)]"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "b11cc8e3",
   "metadata": {},
   "source": [
    "### Yearly airline performance report\n",
    "1. Number of flights under different cancellation categories using bar chart. \n",
    "2. Average flight time by reporting airline using line chart. \n",
    "3. Percentage of diverted airport landings per reporting airline using pie chart. \n",
    "4. Number of flights flying from each state using choropleth map. \n",
    "5. Number of flights flying to each state from each reporting airline using treemap chart.\n",
    "\n",
    "### Yearly average flight delay statistics For the chosen year provide,\n",
    "1. Monthly average carrier delay by reporting airline for the given year. \n",
    "2. Monthly average weather delay by reporting airline for the given year.\n",
    "3. Monthly average national air system delay by reporting airline for the given year. \n",
    "4. Monthly average security delay by reporting airline for the given year. \n",
    "5. Monthly average late aircraft delay by reporting airline for the given year\n",
    "\n",
    "Columns we need :\n",
    "Month, Year, Flights, CancellationCode, Reporting_Airline, 'AirTime','DivAirportLandings','OriginState','DestState','CarrierDelay','WeatherDelay','NASDelay','SecurityDelay','LateAircraftDelay'"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "id": "e5ac59b0",
   "metadata": {},
   "outputs": [],
   "source": [
    "def compute_data_choice_1(df):\n",
    "    #Cancellation Category Count for Bar Chart\n",
    "    bar_data = df.groupby(['Month','CancellationCode'])['Flights'].sum().reset_index()\n",
    "    #Average flight time by reporting airline\n",
    "    line_data = df.groupby(['Month','Reporting_Airline'])['AirTime'].mean().reset_index()\n",
    "    #Diverted Airport Landings\n",
    "    div_data = df[df['DivAirportLandings'] != 0.0]\n",
    "    #Source state count\n",
    "    map_data = df.groupby(['OriginState'])['Flights'].sum().reset_index()\n",
    "    #Destination state count\n",
    "    tree_data = df.groupby(['DestState', 'Reporting_Airline'])['Flights'].sum().reset_index()\n",
    "    return bar_data, line_data, div_data, map_data, tree_data\n",
    "\n",
    "def compute_data_choice_2(df):\n",
    "    #Compute delay average \n",
    "    avg_car = df.groupby(['Month','Reporting_Airline'])['CarrierDelay'].mean().reset_index()\n",
    "    avg_weather = df.groupby(['Month','Reporting_Airline'])['WeatherDelay'].mean().reset_index()\n",
    "    avg_NAS = df.groupby(['Month','Reporting_Airline'])['NASDelay'].mean().reset_index()\n",
    "    avg_sec = df.groupby(['Month','Reporting_Airline'])['SecurityDelay'].mean().reset_index()\n",
    "    avg_late = df.groupby(['Month','Reporting_Airline'])['LateAircraftDelay'].mean().reset_index()\n",
    "    return avg_car, avg_weather, avg_NAS, avg_sec, avg_late "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "id": "a0f09fa6",
   "metadata": {},
   "outputs": [],
   "source": [
    "#Create dash application and clear the layout until callback executed\n",
    "app = dash.Dash(__name__)\n",
    "\n",
    "#Set the application to show the graphs after callback\n",
    "app.config.suppress_callback_exceptions = True"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "bf3cf740",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Dash is running on http://127.0.0.1:8050/\n",
      "\n",
      " * Serving Flask app '__main__' (lazy loading)\n",
      " * Environment: production\n",
      "\u001b[31m   WARNING: This is a development server. Do not use it in a production deployment.\u001b[0m\n",
      "\u001b[2m   Use a production WSGI server instead.\u001b[0m\n",
      " * Debug mode: off\n"
     ]
    },
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      " * Running on http://127.0.0.1:8050/ (Press CTRL+C to quit)\n",
      "127.0.0.1 - - [06/Jan/2022 17:04:20] \"GET / HTTP/1.1\" 200 -\n",
      "127.0.0.1 - - [06/Jan/2022 17:04:21] \"GET /_dash-layout HTTP/1.1\" 200 -\n",
      "127.0.0.1 - - [06/Jan/2022 17:04:21] \"GET /_dash-dependencies HTTP/1.1\" 200 -\n"
     ]
    },
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Exception on /_dash-update-component [POST]\n",
      "Traceback (most recent call last):\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\flask\\app.py\", line 2073, in wsgi_app\n",
      "    response = self.full_dispatch_request()\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\flask\\app.py\", line 1518, in full_dispatch_request\n",
      "    rv = self.handle_user_exception(e)\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\flask\\app.py\", line 1516, in full_dispatch_request\n",
      "    rv = self.dispatch_request()\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\flask\\app.py\", line 1502, in dispatch_request\n",
      "    return self.ensure_sync(self.view_functions[rule.endpoint])(**req.view_args)\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\dash\\dash.py\", line 1078, in dispatch\n",
      "    response.set_data(func(*args, outputs_list=outputs_list))\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\dash\\dash.py\", line 1009, in add_context\n",
      "    output_value = func(*args, **kwargs)  # %% callback invoked %%\n",
      "  File \"C:\\temp/ipykernel_9592/157844534.py\", line 123, in get_graph\n",
      "    carrier_fig = px.line(avg_car, x='Month', y='CarrierDelay', color='Reporting_Airline', title='Average carrrier delay time (minutes) by airline')\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\plotly\\express\\_chart_types.py\", line 252, in line\n",
      "    return make_figure(args=locals(), constructor=go.Scatter)\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\plotly\\express\\_core.py\", line 1889, in make_figure\n",
      "    for val in sorted_group_values[m.grouper]:\n",
      "KeyError: 'Reporting_Airline'\n"
     ]
    },
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "127.0.0.1 - - [06/Jan/2022 17:04:21] \"POST /_dash-update-component HTTP/1.1\" 500 -\n"
     ]
    },
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Exception on /_dash-update-component [POST]\n",
      "Traceback (most recent call last):\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\flask\\app.py\", line 2073, in wsgi_app\n",
      "    response = self.full_dispatch_request()\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\flask\\app.py\", line 1518, in full_dispatch_request\n",
      "    rv = self.handle_user_exception(e)\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\flask\\app.py\", line 1516, in full_dispatch_request\n",
      "    rv = self.dispatch_request()\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\flask\\app.py\", line 1502, in dispatch_request\n",
      "    return self.ensure_sync(self.view_functions[rule.endpoint])(**req.view_args)\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\dash\\dash.py\", line 1078, in dispatch\n",
      "    response.set_data(func(*args, outputs_list=outputs_list))\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\dash\\dash.py\", line 1009, in add_context\n",
      "    output_value = func(*args, **kwargs)  # %% callback invoked %%\n",
      "  File \"C:\\temp/ipykernel_9592/157844534.py\", line 83, in get_graph\n",
      "    bar_fig = px.bar(bar_data, x='Month', y='Flights', color='CancellationCode', title='Monthly Flight Cancellation')\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\plotly\\express\\_chart_types.py\", line 350, in bar\n",
      "    return make_figure(\n",
      "  File \"C:\\Users\\ASUS\\anaconda3\\lib\\site-packages\\plotly\\express\\_core.py\", line 1889, in make_figure\n",
      "    for val in sorted_group_values[m.grouper]:\n",
      "KeyError: 'CancellationCode'\n"
     ]
    },
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "127.0.0.1 - - [06/Jan/2022 17:04:24] \"POST /_dash-update-component HTTP/1.1\" 500 -\n",
      "127.0.0.1 - - [06/Jan/2022 17:04:31] \"POST /_dash-update-component HTTP/1.1\" 200 -\n"
     ]
    }
   ],
   "source": [
    "\n",
    "# Application layout\n",
    "app.layout = html.Div(children=[\n",
    "                                html.H1('US Domestic Airline Flights Performance', \n",
    "                                style={'textAlign': 'center', 'color': '#503D36',\n",
    "                                'font-size': 24}),\n",
    "                                \n",
    "                                #Outer division\n",
    "                                html.Div([\n",
    "                                    #Inner division 1\n",
    "                                    html.Div([\n",
    "                                        html.Div(\n",
    "                                            [\n",
    "                                            html.H2('Report Type:', style={'margin-right': '2em'}),\n",
    "                                            ]\n",
    "                                                ),\n",
    "                                         dcc.Dropdown(id='input-type', \n",
    "                                                    options=[\n",
    "                                                    {'label': 'Yearly Airline Performance Report', 'value': 'OPT1'},\n",
    "                                                    {'label': 'Yearly Airline Delay Report', 'value': 'OPT2'}\n",
    "                                                    ],\n",
    "                                                    placeholder='Select a report type',\n",
    "                                                    style={'width': '80%', 'padding': '3px', 'font size': '20px', 'text-align-last': 'center'})], \n",
    "                                                    style={'display':'flex'}),\n",
    "                                    \n",
    "                                    #Inner division 2\n",
    "                                    html.Div([\n",
    "                                         html.Div(\n",
    "                                            [\n",
    "                                            html.H2('Choose Year:', style={'margin-right': '2em'}),\n",
    "                                            ]\n",
    "                                                ),\n",
    "                                        dcc.Dropdown(id='input-year', \n",
    "                                                     # Update dropdown values using list comphrehension\n",
    "                                                     options=[{'label': i, 'value': i} for i in year_list],\n",
    "                                                     placeholder=\"Select a year\",\n",
    "                                                     style={'width':'80%', 'padding':'3px', 'font-size': '20px', 'text-align-last' : 'center'}),\n",
    "                                            # Place them next to each other using the division style\n",
    "                                            ], style={'display': 'flex'}),  \n",
    "                                          ]),\n",
    "                                \n",
    "                                # Add computer graphs to be updated during callback\n",
    "                                #1st division\n",
    "                                html.Div([ ], id='plot1'),\n",
    "                                #2nd division\n",
    "                                html.Div([\n",
    "                                        html.Div([ ], id='plot2'),\n",
    "                                        html.Div([ ], id='plot3')\n",
    "                                ], style={'display': 'flex'}),\n",
    "                                #3rd division\n",
    "                                html.Div([\n",
    "                                        html.Div([ ], id='plot4'),\n",
    "                                        html.Div([ ], id='plot5')\n",
    "                                ], style={'display': 'flex'})])\n",
    "\n",
    "#Create the callback\n",
    "\n",
    "@app.callback([Output(component_id='plot1', component_property='children'),\n",
    "               Output(component_id='plot2', component_property='children'),\n",
    "               Output(component_id='plot3', component_property='children'),\n",
    "               Output(component_id='plot4', component_property='children'),\n",
    "               Output(component_id='plot5', component_property='children')],\n",
    "              [Input(component_id='input-type', component_property='value'),\n",
    "               Input(component_id='input-year', component_property='value')],\n",
    "               #Holding output state till user enters all the form information (type of chart and year)\n",
    "              [State(\"plot1\", 'children'), State(\"plot2\", \"children\"),\n",
    "               State(\"plot3\", \"children\"), State(\"plot4\", \"children\"),\n",
    "               State(\"plot5\", \"children\")\n",
    "              ])\n",
    "\n",
    "\n",
    "\n",
    "# Add computation to callback function and return graph\n",
    "def get_graph(chart, year, children1, children2, c3, c4, c5):\n",
    "      \n",
    "        # Select data\n",
    "        df =  airline_data[airline_data['Year']== year]\n",
    "       \n",
    "        if chart == 'OPT1':\n",
    "            # Compute required information for creating graph from the data\n",
    "            bar_data, line_data, div_data, map_data, tree_data = compute_data_choice_1(df)\n",
    "            \n",
    "            # Number of flights under different cancellation categories\n",
    "            bar_fig = px.bar(bar_data, x='Month', y='Flights', color='CancellationCode', title='Monthly Flight Cancellation')\n",
    "            \n",
    "            # Average flight time by reporting airline\n",
    "            line_fig = px.line(line_data, x ='Month', y='AirTime', color = 'Reporting_Airline', title = 'Average monthly flight time (minutes) by airline.')\n",
    "            \n",
    "            # Percentage of diverted airport landings per reporting airline\n",
    "            pie_fig = px.pie(div_data, values='Flights', names='Reporting_Airline', title='% of flights by reporting airline')\n",
    "            \n",
    "            # Number of flights flying from each state using choropleth\n",
    "            map_fig = px.choropleth(map_data,  # Input data\n",
    "                    locations='OriginState', \n",
    "                    color='Flights',  \n",
    "                    hover_data=['OriginState', 'Flights'], \n",
    "                    locationmode = 'USA-states', # Set to plot as US States\n",
    "                    color_continuous_scale='GnBu',\n",
    "                    range_color=[0, map_data['Flights'].max()]) \n",
    "            map_fig.update_layout(\n",
    "                    title_text = 'Number of flights from origin state', \n",
    "                    geo_scope='usa') # Plot only the USA instead of globe\n",
    "            \n",
    "            # Number of flights flying to each state from each reporting airline\n",
    "            tree_fig = px.treemap(tree_data, path=['DestState', 'Reporting_Airline'], \n",
    "                      values='Flights',\n",
    "                      color='Reporting_Airline',\n",
    "                      color_continuous_scale='RdBu',\n",
    "                      title='Flight count by airline to destination state'\n",
    "                )\n",
    "            return [dcc.Graph(figure=tree_fig), \n",
    "                    dcc.Graph(figure=pie_fig),\n",
    "                    dcc.Graph(figure=map_fig),\n",
    "                    dcc.Graph(figure=bar_fig),\n",
    "                    dcc.Graph(figure=line_fig)\n",
    "                   ]\n",
    "            \n",
    "        else:\n",
    "            # Charts under \"Flight Delay Time Statistics Dashboard\" section\n",
    "            # Compute required information for creating graph from the data\n",
    "            avg_car, avg_weather, avg_NAS, avg_sec, avg_late = compute_data_choice_2(df)\n",
    "            \n",
    "            # Create graph\n",
    "            carrier_fig = px.line(avg_car, x='Month', y='CarrierDelay', color='Reporting_Airline', title='Average carrrier delay time (minutes) by airline')\n",
    "            weather_fig = px.line(avg_weather, x='Month', y='WeatherDelay', color='Reporting_Airline', title='Average weather delay time (minutes) by airline')\n",
    "            nas_fig = px.line(avg_NAS, x='Month', y='NASDelay', color='Reporting_Airline', title='Average NAS delay time (minutes) by airline')\n",
    "            sec_fig = px.line(avg_sec, x='Month', y='SecurityDelay', color='Reporting_Airline', title='Average security delay time (minutes) by airline')\n",
    "            late_fig = px.line(avg_late, x='Month', y='LateAircraftDelay', color='Reporting_Airline', title='Average late aircraft delay time (minutes) by airline')\n",
    "            \n",
    "            return[dcc.Graph(figure=carrier_fig), \n",
    "                   dcc.Graph(figure=weather_fig), \n",
    "                   dcc.Graph(figure=nas_fig), \n",
    "                   dcc.Graph(figure=sec_fig), \n",
    "                   dcc.Graph(figure=late_fig)]\n",
    "        \n",
    "# Run the app\n",
    "if __name__ == '__main__':\n",
    "    app.run_server()"
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3 (ipykernel)",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.8.5"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 5
}
