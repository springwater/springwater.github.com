---
layout: post
title: "数据库脚本导出工具"
tagline: "支持mysql、oracle、mssql、db2，附源码"
description: ""
category : code
tags: [java, database, mysql, oracle, mssql, db2]
---
{% include JB/setup %}
<script type="text/javascript" src="/assets/highslide/highslide.js"></script>
<link rel="stylesheet" type="text/css" href="/assets/highslide/highslide.css" />

<script type="text/javascript">
	hs.graphicsDir = '/assets/highslide/graphics/';
	hs.wrapperClassName = 'wide-border';
</script>

###源码
{% highlight java %}
package com.sw.tools;

import java.io.BufferedReader;

public class Export {

	protected Shell shell;
	private Combo database;
	private Text driver;
	private Text url;
	private Text user;
	private Text password;
	private Button btnNewButton;
	private Button btnNewButton_1;
	private Text sqlfile;
	private Text test;
	private String sel_file;
	private String dbtype;
	private String expdb;
	private Connection conn;
	private Combo expdb_combo;
	private Label expdb_label;

	/**
	 * Launch the application.
	 * 
	 * @param args
	 */
	public static void main(String[] args) {
		try {
			Export window = new Export();
			window.open();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * Open the window.
	 */
	public void open() {
		Display display = Display.getDefault();
		createContents();
		shell.open();
		shell.layout();
		while (!shell.isDisposed()) {
			if (!display.readAndDispatch()) {
				display.sleep();
			}
		}
	}

	/**
	 * Create contents of the window.
	 */
	protected void createContents() {
		shell = new Shell();
		shell.setSize(600, 360);
		shell.setText("数据库脚本导出工具");

		Label lblNewLabel = new Label(shell, SWT.NONE);
		lblNewLabel.setBounds(28, 54, 61, 17);
		lblNewLabel.setText("Driver:");

		Label lblNewLabel_1 = new Label(shell, SWT.NONE);
		lblNewLabel_1.setBounds(28, 101, 61, 17);
		lblNewLabel_1.setText("Url:");

		Label lblNewLabel_2 = new Label(shell, SWT.NONE);
		lblNewLabel_2.setBounds(28, 140, 61, 17);
		lblNewLabel_2.setText("User:");

		Label lblNewLabel_3 = new Label(shell, SWT.NONE);
		lblNewLabel_3.setBounds(28, 183, 61, 17);
		lblNewLabel_3.setText("Password:");

		Label lblNewLabel_4 = new Label(shell, SWT.NONE);
		lblNewLabel_4.setBounds(28, 226, 61, 17);
		lblNewLabel_4.setText("Sql File:");

		database = new Combo(shell, SWT.NONE);
		database.addSelectionListener(new SelectionAdapter() {
			@Override
			public void widgetSelected(SelectionEvent e) {
				if ("MYSQL".equals(database.getText())) {
					driver.setText("com.mysql.jdbc.Driver");
					url.setText("jdbc:mysql://localhost:3306/sw");
					user.setText("root");
					password.setText("password");
				} else if ("ORACLE".equals(database.getText())) {
					driver.setText("oracle.jdbc.driver.OracleDriver");
					url.setText("jdbc:oracle:thin:@localhost:1521:sw");
					user.setText("system");
					password.setText("password");
				} else if ("MSSQL".equals(database.getText())) {
					driver.setText("com.microsoft.sqlserver.jdbc.SQLServerDriver");
					url.setText("jdbc:sqlserver://localhost:1433;DatabaseName=sw");
					user.setText("sa");
					password.setText("password");
				} else if ("DB2".equals(database.getText())) {
					driver.setText("com.ibm.db2.jcc.DB2Driver");
					url.setText("jdbc:db2://localhost:50000/sw");
					user.setText("system");
					password.setText("password");
				}
			}
		});
		database.setItems(new String[] { "MYSQL", "ORACLE", "MSSQL", "DB2" });
		database.setBounds(108, 10, 88, 25);
		database.setText("MYSQL");

		driver = new Text(shell, SWT.BORDER);
		driver.setText("com.mysql.jdbc.Driver");
		driver.setBounds(108, 48, 300, 23);

		url = new Text(shell, SWT.BORDER);
		url.setText("jdbc:mysql://localhost:3306/sw");
		url.setBounds(108, 95, 300, 23);

		user = new Text(shell, SWT.BORDER);
		user.setText("root");
		user.setBounds(108, 134, 300, 23);

		password = new Text(shell, SWT.BORDER | SWT.PASSWORD);
		password.setText("password");
		password.setBounds(108, 177, 300, 23);

		sqlfile = new Text(shell, SWT.BORDER | SWT.READ_ONLY);
		sqlfile.setBounds(108, 220, 300, 23);

		test = new Text(shell, SWT.BORDER | SWT.READ_ONLY);
		test.setBounds(208, 279, 73, 23);

		btnNewButton = new Button(shell, SWT.NONE);
		btnNewButton.addSelectionListener(new SelectionAdapter() {
			@Override
			public void widgetSelected(SelectionEvent e) {
				dbtype = database.getText();
				expdb = expdb_combo.getText();
				String sdriver = driver.getText();
				String surl = url.getText();
				String suser = user.getText();
				String spassword = password.getText();
				boolean conn_bool = false;
				try {
					Class.forName(sdriver).newInstance();
					conn = DriverManager.getConnection(surl, suser, spassword);
					conn_bool = true;
				} catch (Exception conn_e) {
				}
				System.out.println("dbtype=" + dbtype);
				System.out.println("conn=" + conn);
				System.out.println("expdb=" + expdb);
				test.setText(conn_bool + "");
				MessageBox box = new MessageBox(shell);
				if (conn_bool) {
					box.setText("连接成功。");
					box.setMessage("连接成功。");
				} else {
					box.setText("连接失败。");
					box.setMessage("连接失败。");
				}
				box.open();
			}
		});
		btnNewButton.setBounds(28, 275, 80, 27);
		btnNewButton.setText("Test");

		btnNewButton_1 = new Button(shell, SWT.NONE);
		btnNewButton_1.addSelectionListener(new SelectionAdapter() {
			@Override
			public void widgetSelected(SelectionEvent e) {
				MessageBox box = new MessageBox(shell);
				if (conn != null && sel_file != null && !"".equals(sel_file)) {
					FileDialog dialog = new FileDialog(shell, SWT.SAVE);
					dialog.setFilterNames(new String[] { "Sql Files", "All Files (*.*)" });
					dialog.setFilterExtensions(new String[] { "*.sql", "*.*" });
					dialog.setFileName("exp.sql");
					boolean done = false;
					while (!done) {
						String exp_file = dialog.open();
						if (exp_file != null) {
							if (exp(conn, sel_file, exp_file)) {
								box.setMessage("导出成功。");
								box.open();
							}
						}
						done = true;
					}
				} else {
					if (conn == null) {
						box.setMessage("请先测试数据库连接。");
						box.open();
						return;
					}
					if (sel_file == null || "".equals(sel_file)) {
						box.setMessage("请选择查询的sql文件。");
						box.open();
					}
				}
			}
		});
		btnNewButton_1.setBounds(122, 275, 80, 27);
		btnNewButton_1.setText("Export");

		Button btnNewButton_2 = new Button(shell, SWT.NONE);
		btnNewButton_2.addSelectionListener(new SelectionAdapter() {
			@Override
			public void widgetSelected(SelectionEvent e) {
				FileDialog dialog = new FileDialog(shell, SWT.OK);
				dialog.setFilterNames(new String[] { "Sql Files", "All Files (*.*)" });
				dialog.setFilterExtensions(new String[] { "*.sql", "*.*" });
				dialog.setFileName("select.sql");
				String fileName = null;
				boolean done = false;
				while (!done) {
					sel_file = dialog.open();
					fileName = sel_file;
					if (fileName == null) {
						done = true;
					} else {
						File file = new File(fileName);
						if (file.exists()) {
							sqlfile.setText(sel_file);
							done = true;
						} else {
							MessageBox mb = new MessageBox(dialog.getParent(), SWT.ICON_WARNING | SWT.YES | SWT.NO);
							mb.setMessage(fileName + " 不存在，请重新选择。");
							done = mb.open() == SWT.YES;
							done = false;
						}
					}
				}
			}
		});
		btnNewButton_2.setBounds(439, 216, 80, 27);
		btnNewButton_2.setText("浏览");

		Label lblNewLabel_5 = new Label(shell, SWT.NONE);
		lblNewLabel_5.setBounds(28, 10, 74, 17);
		lblNewLabel_5.setText("数据库链接:");

		expdb_combo = new Combo(shell, SWT.NONE);
		expdb_combo.setItems(new String[] { "MYSQL", "ORACLE", "MSSQL", "DB2" });
		expdb_combo.setBounds(304, 10, 88, 25);
		expdb_combo.setText("MYSQL");

		expdb_label = new Label(shell, SWT.NONE);
		expdb_label.setText("导出数据库:");
		expdb_label.setBounds(224, 10, 74, 17);

	}

	private boolean exp(Connection conn, String sel_file, String exp_file) {
		SimpleDateFormat sfd = new SimpleDateFormat("yyyy-MM-dd");
		SimpleDateFormat sft = new SimpleDateFormat("HH:mm:ss");
		PreparedStatement ps = null;
		ResultSet rs = null;
		try {
			File file = new File(exp_file);
			file.createNewFile();
			FileOutputStream fos = new FileOutputStream(file);
			PrintWriter pw = new PrintWriter(fos);
			FileReader fr = new FileReader(sel_file);
			BufferedReader r = new BufferedReader(fr);
			String line;
			while ((line = r.readLine()) != null) {
				if (line != null && !"".equals(line.trim())) {
					try {
						if (line.endsWith(";")) {
							line = line.substring(0, line.length() - 1);
							ps = conn.prepareStatement(line);
							rs = ps.executeQuery();
							ResultSetMetaData rsmd = rs.getMetaData();
							String tableName = rsmd.getTableName(1);
							if ("MSSQL,ORACLE".contains(dbtype)) {
								String tmp = StringUtils.substringBetween(line, "FROM", "WHERE");
								if (tmp == null) {
									line = line + ";";
									tmp = StringUtils.substringBetween(line, "FROM", ";");
									if (tmp != null) {
										tableName = tmp.trim();
									}
								} else {
									tableName = tmp.trim();
								}
							}
							if (StringUtils.isNotBlank(tableName)) {
								while (rs.next()) {
									StringBuilder sql = new StringBuilder();
									sql.append("INSERT INTO " + tableName + " (");
									String fstr = "";
									for (int i = 1; i <= rsmd.getColumnCount(); i++) {
										fstr += "," + rsmd.getColumnLabel(i);
									}
									fstr = fstr.substring(1);
									sql.append(fstr);
									sql.append(")");
									sql.append(" VALUES (");
									String vstr = "";
									for (int i = 1; i <= rsmd.getColumnCount(); i++) {
										if (rs.getString(rsmd.getColumnLabel(i)) == null) {
											vstr += ",NULL";
										} else {
											if ("VARCHAR,VARCHAR2".contains(rsmd.getColumnTypeName(i).toUpperCase())) {
												vstr += ",'" + rs.getString(rsmd.getColumnLabel(i)) + "'";
											} else if ("TINYINT,INT,INTEGER,BIGINT,DOUBLE,NUMBER,NUMERIC,FLOAT"
													.contains(rsmd.getColumnTypeName(i).toUpperCase())) {
												vstr += "," + rs.getInt(rsmd.getColumnLabel(i)) + "";
											} else if ("DATETIME,TIMESTAMP".equals(rsmd.getColumnTypeName(i)
													.toUpperCase())) {
												if ("ORACLE".equals(expdb)) {
													vstr += ",to_date('"
															+ sfd.format(rs.getDate(rsmd.getColumnLabel(i))) + " "
															+ sft.format(rs.getTime(rsmd.getColumnLabel(i)))
															+ "','yyyy-mm-dd hh24:mi:ss')";
												} else {
													vstr += ",'" + sfd.format(rs.getDate(rsmd.getColumnLabel(i))) + " "
															+ sft.format(rs.getTime(rsmd.getColumnLabel(i))) + "'";
												}
											} else {
												vstr += ",'" + rs.getString(rsmd.getColumnLabel(i)) + "'";
											}
										}
									}
									sql.append(vstr.substring(1));
									sql.append(");");
									System.out.println(sql.toString());
									sql.append("\r\n");
									pw.write(sql.toString().toCharArray());
								}
								System.out.println();
								pw.write("\r\n");
							}
						} else {
							System.err.println(line);
						}
					} catch (Exception e) {
						e.printStackTrace();
					}
				}
			}
			pw.flush();
			pw.close();
			test.setText("false");
			return true;
		} catch (Exception e) {
			e.printStackTrace();
			return false;
		} finally {
			try {
				if (rs != null) {
					rs.close();
				}
				if (ps != null) {
					ps.close();
				}
				if (conn != null) {
					conn.close();
					conn = null;
				}
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
}

{% endhighlight %}

###有图有真相

<a href="/assets/img/130528_export.jpg" class="highslide" onclick="return hs.expand(this)">
<img src="/assets/img/130528_export.jpg" alt="Highslide JS" title="Click to enlarge" height="600" width="360" /></a>
