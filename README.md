import React, { useEffect, useState } from "react";
import axios from "axios";
import { PieChart, Pie, Cell, Tooltip, Legend } from "recharts";
import { CSVLink } from "react-csv";

const AdminDashboard = () => {
    const [users, setUsers] = useState([]);
    const [stats, setStats] = useState(null);

    useEffect(() => {
        fetchUsers();
        fetchStats();
    }, []);

    const fetchUsers = async () => {
        try {
            const response = await axios.get("https://your-backend.onrender.com/admin/users", {
                headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
            });
            setUsers(response.data);
        } catch (error) {
            console.error("Error fetching users", error);
        }
    };

    const fetchStats = async () => {
        try {
            const response = await axios.get("https://your-backend.onrender.com/admin/stats", {
                headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
            });
            setStats(response.data);
        } catch (error) {
            console.error("Error fetching stats", error);
        }
    };

    const updateRole = async (id, role) => {
        try {
            await axios.put(`https://your-backend.onrender.com/admin/users/${id}`, { role }, {
                headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
            });
            logAction(`Updated role to ${role}`, id);
            sendNotification(users.find(user => user.id === id).email, "Role Update", `Your role has been updated to ${role}.`);
            fetchUsers();
            fetchStats();
        } catch (error) {
            console.error("Error updating role", error);
        }
    };

    const deleteUser = async (id) => {
        if (!window.confirm("Are you sure you want to delete this user?")) return;
        try {
            await axios.delete(`https://your-backend.onrender.com/admin/users/${id}`, {
                headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
            });
            logAction("Deleted user", id);
            sendNotification(users.find(user => user.id === id).email, "Account Deleted", "Your account has been deleted by the admin.");
            fetchUsers();
            fetchStats();
        } catch (error) {
            console.error("Error deleting user", error);
        }
    };

    const logAction = async (action, userId) => {
        try {
            await axios.post("https://your-backend.onrender.com/admin/logs", { action, userId }, {
                headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
            });
        } catch (error) {
            console.error("Error logging action", error);
        }
    };

    const sendNotification = async (email, subject, message) => {
        try {
            await axios.post("https://your-backend.onrender.com/admin/notify", { email, subject, message }, {
                headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
            });
        } catch (error) {
            console.error("Error sending notification", error);
        }
    };

    const pieData = stats ? [
        { name: "Admins", value: stats.adminCount },
        { name: "Moderators", value: stats.moderatorCount },
        { name: "Editors", value: stats.editorCount },
        { name: "Users", value: stats.userCount }
    ] : [];

    return (
        <div className="container mx-auto p-4">
            <h2 className="text-2xl font-bold mb-4 text-center">Admin Dashboard</h2>
            <div className="flex justify-between mb-4">
                <CSVLink data={users} filename="users.csv" className="bg-blue-500 text-white px-4 py-2">Export CSV</CSVLink>
            </div>
            {stats && (
                <div className="mb-4 p-4 border rounded bg-gray-100">
                    <h3 className="text-lg font-semibold">User Statistics</h3>
                    <p>Total Users: {stats.totalUsers}</p>
                    <p>New Users (Last 7 Days): {stats.newUsers}</p>
                    <PieChart width={400} height={300}>
                        <Pie data={pieData} dataKey="value" nameKey="name" cx="50%" cy="50%" outerRadius={100} fill="#8884d8" label>
                            {pieData.map((entry, index) => (
                                <Cell key={`cell-${index}`} fill={["#8884d8", "#82ca9d", "#ffc658", "#ff8042"][index]} />
                            ))}
                        </Pie>
                        <Tooltip />
                        <Legend />
                    </PieChart>
                </div>
            )}
            <table className="min-w-full bg-white border border-gray-300">
                <thead>
                    <tr className="bg-gray-200">
                        <th className="border px-4 py-2">Name</th>
                        <th className="border px-4 py-2">Email</th>
                        <th className="border px-4 py-2">Role</th>
                        <th className="border px-4 py-2">Actions</th>
                    </tr>
                </thead>
                <tbody>
                    {users.map((user) => (
                        <tr key={user.id} className="hover:bg-gray-100">
                            <td className="border px-4 py-2">{user.name}</td>
                            <td className="border px-4 py-2">{user.email}</td>
                            <td className="border px-4 py-2">
                                <select value={user.role} onChange={(e) => updateRole(user.id, e.target.value)}>
                                    <option value="user">User</option>
                                    <option value="editor">Editor</option>
                                    <option value="moderator">Moderator</option>
                                    <option value="admin">Admin</option>
                                </select>
                            </td>
                            <td className="border px-4 py-2">
                                <button className="bg-red-500 text-white px-4 py-2" onClick={() => deleteUser(user.id)}>Delete</button>
                            </td>
                        </tr>
                    ))}
                </tbody>
            </table>
        </div>
    );
};

export default AdminDashboard;
